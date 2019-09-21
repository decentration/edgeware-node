This guide covers how to set up a public Edgeware node. In addition to 
running an up-to-date version of the software, public nodes have a few more
requirements:

1. The node should run as a system service, automatically starting on boot
2. A secure WebSockets connection, which requires a proxy (NGINX)
3. A signed HTTPS certificate, issued by a recognized CA (Let's Encrypt)

(Optional: System monitoring software to reboot the node automatically when
it stalls or loses connectivity. Those instructions will be added later.)

## 0. Provisioning a server

Provision an appropriately sized server from one of the recommended VPS
providers.

We recommend 2GB or 4GB of RAM and assume you are using Ubuntu 18.04 x64.
Other versions or operating systems will require adjustments to these instructions.

Set up DNS from a domain name that you own to point to the server. We will use
`testnet1.edgewa.re`. (You don't need to do this if you are setting up a private node.)

SSH into the server.

## 1. Installing Edgeware and setting it up as a system service

First, clone the `edgeware-node` repo, install any dependencies, and run
the required build scripts.

```
apt update
apt install -y gcc libc6-dev
apt install -y cmake pkg-config libssl-dev git clang libclang-dev

# Prefetch SSH publickeys
ssh-keyscan -H github.com >> ~/.ssh/known_hosts

# Install rustup
curl https://sh.rustup.rs -sSf | sh -s -- -y
source /root/.cargo/env
export PATH=/root/.cargo/bin:$PATH

# Get packages
git clone https://github.com/hicommonwealth/edgeware-node.git
cd edgeware-node

# Build packages
./setup.sh
```

Set up the node as a system service. To do this, navigate into the root directory
of the `edgeware-node` repo and execute the following to create the service
configuration file:

```
{
    echo '[Unit]'
    echo 'Description=Edgeware'
    echo '[Service]'
    echo 'Type=exec'
    echo 'WorkingDirectory='`pwd`
    echo 'ExecStart='`pwd`'/target/release/edgeware --chain=edgeware --ws-external --rpc-cors "*"'
    echo '[Install]'
    echo 'WantedBy=multi-user.target'
} > /etc/systemd/system/edgeware.service
```

**Note: This will create an Edgeware server that accepts all incoming
connections. This is risky and insecure if you keep any keys on the server.
If you would like to create a private node instead, you should remove the
`ws-external` and `rpc-cors` flags.**

Double check that the config has been written to `/etc/systemd/system/edgeware.service`
correctly. If so, enable the service so it runs on startup, and then try to
start it now:

```
systemctl enable edgeware
systemctl start edgeware
```

Check the status of the service:

```
systemctl status edgeware
```

You should see the node connecting to the network and syncing the latest blocks.
If you need to tail the latest output, you can use:

```
journalctl -u edgeware.service
```

## 2. Configuring an SSL certificate (public nodes only)

We will use Certbot to talk to Let's Encrypt. Install Certbot dependencies:

```
apt -y install software-properties-common
add-apt-repository universe
add-apt-repository ppa:certbot/certbot
apt update
```

Install Certbot:

```
apt -y install certbot python-certbot-nginx
```

It will guide you through getting a certificate from Let's Encrypt:

```
certbot certonly --standalone
```

If you already have a web server running (e.g. nginx, Apache, etc.)
you will need to stop it, by running e.g. `service nginx stop`, for
this to work.

Certbot will ask you some questions, start its own web server, and
talk to Let's Encrypt to issue a certificate. In the end, you should
see output that looks like this:

```
root:~/edgeware-node# certbot certonly --standalone
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator standalone, Installer None
Please enter in your domain name(s) (comma and/or space separated)  (Enter 'c'
to cancel): testnet1.edgewa.re
Obtaining a new certificate
Performing the following challenges:
http-01 challenge for testnet1.edgewa.re
Waiting for verification...
Cleaning up challenges

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/testnet1.edgewa.re/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/testnet1.edgewa.re/privkey.pem
   Your cert will expire on 2019-10-08. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot
   again. To non-interactively renew *all* of your certificates, run
   "certbot renew"
```

## 3. Configuring a Websockets proxy

First, install nginx:

```
apt -y install nginx
```

Set the intended public address of the server, e.g.
`testnet1.edgewa.re`, as an environment variable:

```
export name=testnet1.edgewa.re
```

Set up an nginx configuration. This will inject the public
address you have just defined.

```
{
    echo 'user       www-data;  ## Default: nobody'
    echo 'worker_processes  5;  ## Default: 1'
    echo 'error_log  /var/log/nginx/error.log;'
    echo 'pid        /var/run/nginx.pid;'
    echo 'worker_rlimit_nofile 8192;'
    echo ''
    echo 'events {'
    echo '  worker_connections  4096;  ## Default: 1024'
    echo '}'
    echo ''
    echo 'http {'
    echo '    map $http_upgrade $connection_upgrade {'
    echo '      default upgrade;'
    echo "      \'\' close;"
    echo '    }'
    echo '    server {'
    echo '      listen       443 ssl;'
    echo '      server_name  '$name';'
    echo ''
    echo '      ssl_certificate /etc/letsencrypt/live/'$name'/cert.pem;'
    echo '      ssl_certificate_key /etc/letsencrypt/live/'$name'/privkey.pem;'
    echo '      ssl_session_timeout 5m;'
    echo '      ssl_protocols  SSLv2 SSLv3 TLSv1;'
    echo '      ssl_ciphers  HIGH:!aNULL:!MD5;'
    echo '      ssl_prefer_server_ciphers   on;'
    echo ''
    echo '      location / {'
    echo '          proxy_pass http://127.0.0.1:9944 ;'
    echo '          proxy_http_version 1.1;'
    echo '          proxy_set_header Upgrade $http_upgrade;'
    echo '          proxy_set_header Connection $connection_upgrade;'
    echo '      }'
    echo '    }'
    echo '}'
} > /etc/nginx/nginx.conf
```

Make sure that the paths of `ssl_certificate` and
`ssl_certificate_key` match what Let's Encrypt produced earlier.
Check that the configuration file has been created correctly.

```
cat /etc/nginx/nginx.conf
nginx -t
```

If there is an error, `nginx -t` should tell you where it is.
**Note that there may be subtle variations in how different
systems are configured, e.g. some boxes may have different
login users or locations for log files. It is up to you to
reconcile these differences.**

Start the server:

```
service nginx restart
```

You can now try to connect to your new node from
[polkadot.js/apps](https://polkadot.js.org/apps/#/settings),
or by making a curl request that emulates opening a secure
WebSockets connection:

```
curl --include --no-buffer --header "Connection: Upgrade" --header "Upgrade: websocket" --header "Host: $name:80" --header "Origin: http://$name:80" --header "Sec-WebSocket-Key: SGVsbG8sIHdvcmxkIQ==" --header "Sec-WebSocket-Version: 13" http://$name:9944/
```

