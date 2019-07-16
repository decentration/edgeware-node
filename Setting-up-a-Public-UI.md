## 0. Provisioning a server

Provision an appropriately sized server from one of the recommended VPS providers.

We assume you are using Ubuntu 18.04 x64; other versions or operating systems will require adjustments to these instructions.

Set up DNS pointing to the server, from e.g. apps.edgewa.re. It is strongly recommended that you do this now.

SSH into the server.

## 1. Installing `apps` and setting it up as a system service

Clone the `apps` repo:

```
git clone https://github.com/hicommonwealth/apps.git
cd apps
```

Install nodejs 11.x and yarn. You will need to add new package repositories:
```
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
curl -sL https://deb.nodesource.com/setup_11.x | sudo -E bash -
apt update
apt install -y nodejs yarn
```

Install dependencies:
```
yarn
```

Create the apps service:
```
{
    echo '[Unit]'
    echo 'Description=EdgewareApps'
    echo '[Service]'
    echo 'Type=exec'
    echo 'WorkingDirectory='`pwd`
    echo 'Environment=ENV=production'
    echo 'ExecStart=yarn run start'
    echo '[Install]'
    echo 'WantedBy=multi-user.target'
} > /etc/systemd/system/apps.service
```

Start and check the service is running:
```
systemctl start apps
systemctl status apps
curl localhost:3000
```

## 2. Configuring an SSL certificate

To make `apps` available in a secure manner, we will use Let's Encrypt
and certbot to make the server available via SSL.

Install Certbot:

```
apt -y install software-properties-common
add-apt-repository universe
add-apt-repository ppa:certbot/certbot
apt update
apt -y install certbot python-certbot-nginx
```

Run certbot to get a certificate from Let's Encrypt:

```
certbot --nginx
```

Certbot will ask you some questions, start its own web
server, and talk to Let's Encrypt to issue a certificate.
It will also configure nginx.

## 3. Updating the nginx configuration

Ensure nginx proxies from port 443 to port 3000:

```
nano /etc/nginx/nginx.conf
```

Replace the configuration file with the following:

```
user       www-data;  ## Default: nobody
worker_processes  5;  ## Default: 1
error_log  /var/log/nginx/error.log;
pid        /var/run/nginx.pid;
worker_rlimit_nofile 8192;

events {
  worker_connections  4096;  ## Default: 1024
}

http {
    map $http_upgrade $connection_upgrade {
    	default upgrade;
    	'' close;
    }
    server {
	listen       443 ssl;
	server_name  apps.edgewa.re;
        ssl_certificate /etc/letsencrypt/live/testnode.edgewa.re/fullchain.pem; # managed by Certbot
        ssl_certificate_key /etc/letsencrypt/live/testnode.edgewa.re/privkey.pem; # managed by Certbot
	ssl_session_timeout 5m;
	ssl_protocols  SSLv2 SSLv3 TLSv1;
	ssl_ciphers  HIGH:!aNULL:!MD5;
	ssl_prefer_server_ciphers   on;

        location / {
            proxy_pass http://127.0.0.1:3000;
        }
    }
}
```
