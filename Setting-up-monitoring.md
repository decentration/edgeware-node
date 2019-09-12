This tutorial explains how to set up monit and mmonit, which are respectively
a process monitoring tool and a dashboard for managing a fleet of monit nodes.

The tutorial assumes you are running Ubuntu 18.04

## Setting up an `mmonit` dashboard server

For the monitoring server, a small server should be enough (e.g. 1GB memory).
The default setup will use SQLite as a database. Beware that if you remove
the monit directory, all logged events will be lost!

```
apt install -y monit
wget https://mmonit.com/dist/mmonit-3.7.3-linux-x64.tar.gz
tar -vxzf mmonit-3.7.3-linux-x64.tar.gz
mv mmonit-3.7.3 monit
```

Use monit to keep mmonit up:

```
{
    echo 'check process mmonit with pidfile '`pwd`'/mmonit/logs/mmonit.pid'
    echo '  start program = "'`pwd`'/mmonit/bin/mmonit"'
    echo '  stop program = "'`pwd`'/mmonit/bin/mmonit stop"'
    echo 'set httpd port 2812 and use address localhost'
    echo '  allow localhost'
} > /etc/monit/monitrc

mmonit reload
mmonit validate
```

To start the mmonit dashboard manually, you can use the command `mmonit/bin/mmonit`.
To stop, use `mmonit/bin/mmonit stop`.

Now, go to the node in your browser, e.g `http://monitor.edgewa.re:8080/`.
Log in with username `admin` and password `swordfish` and change the default
username and password.

You should end up with two users; admin is what you'll use to log in, and the
other user is what you'll provide to nodes that push data to the monitoring server.

## Adding individual nodes

Before proceeding, set an appropriate hostname for the node, e.g.

```
hostnamectl set-hostname validator1
```

Install monit.

```
apt install -y monit
```

Set up a Slack webhook (optional):

```
export WEBHOOK=[slack webhook url]
{
    echo 'URL="'$WEBHOOK'"'
    echo "PAYLOAD='{\"text\": \"`hostname` restarted edgeware.service\"}'"
    echo 'curl -s -X POST --data-urlencode "payload=$PAYLOAD" $URL'
} > /root/slack_notify.sh

chmod 755 /root/slack_notify.sh
```

Set up a Monit config to check the Edgeware service at 10-second intervals,
restart if CPU > 90% for five checks (~50sec), and post events to mmonit.

You should first provide a username and password. They should be the same
as you have set for mmonit above, and they will be used BOTH to push data
to mmonit, AND to allow other hosts to inspect the status of this node
directly.

```
export USER=edgeware
export PASSWORD=[password]
export TARGET=[domain]
{
    echo 'set daemon 10'
    echo 'set log /var/log/monit.log'
    echo 'set idfile /var/lib/monit/id'
    echo 'set statefile /var/lib/monit/state'
    echo 'set eventqueue'
    echo '  basedir /var/lib/monit/events'
    echo '  slots 100                    '
    echo ''
    echo 'check process edgeware matching target/release/edgeware'
    echo '  start program = "/bin/systemctl restart edgeware"'
    echo '  stop program = "/bin/systemctl kill edgeware"'
    echo '  if cpu > 90% for 20 cycles then exec "/bin/systemctl stop edgeware" and repeat every 10 cycles'
    echo '  if cpu > 90% for 64 cycles then exec "/bin/systemctl kill edgeware" and repeat every 10 cycles'
    echo '  if cpu > 90% for 64 cycles then alert'
    echo '  if does not exist for 1 cycles then start'
    echo '  if does not exist for 1 cycles then exec "/bin/bash -c /root/slack_notify.sh"'
    echo ''
    echo 'set mmonit http://'$USER:$PASSWORD@$TARGET':8080/collector'
    echo 'set httpd port 2812 and use address localhost'
    echo '  allow localhost'
    echo '  allow '$USER:$PASSWORD
} > /etc/monit/monitrc

monit reload
monit validate
```

Note that connections on port 2812 are restricted to localhost.

## Bonus: Monitoring API connectivity

It may be beneficial to have monit directly checking the health
of the node by requesting data from it to ensure blocks are syncing,
rather than just checking CPU usage. If your node has an open
WebSockets API, you can use `nodeup` to do this.

Clone [nodeup](https://github.com/hicommonwealth/nodeup.git) into
the `/root` directory. Follow its installation instructions:

```
apt install -y nodejs npm
npm install -g yarn
yarn
```

Add these lines to `/etc/monit/monitrc`:

```
check program nodeup with path "/root/nodeup/index.js"
  if status > 0 for 5 cycles then exec "/bin/systemctl stop edgeware" and repeat every 10 cycles
```

Restart monit, and check that the new script is working:

```
monit reload
monit status
```
