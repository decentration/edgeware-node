## Setting up an `mmonit` dashboard server

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

To start the mmonit dashboard, use `mmonit/bin/mmonit`.
To stop, use `mmonit/bin/mmonit stop`.

Go to the node in your browser, e.g `http://monitor.edgewa.re:8080/`.
Log in with username `admin` and password `swordfish`. Change the default
username and passwords.

## Adding individual nodes

Install monit.

```
apt install -y monit
```


Set up a Monit config to check Edgeware service at 10-second intervals,
and restart if CPU > 80% for five checks, and post events to mmonit.

You should first provide a username and password. They should be the same
as you have set for mmonit above, and they will be used BOTH to push data
to mmonit, AND to allow other hosts to inspect the status of this node
directly.

```
export USER=edgeware
export PASSWORD=f8eec78fe63efae6d8b3ba304cb8d89b
export TARGET=monitor.edgewa.re
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
    echo '  if cpu > 80% for 5 cycles then restart'
    echo '  if cpu > 80% for 5 cycles then alert'
    echo ''
    echo 'set mmonit http://'$USER:$PASSWORD@$TARGET':8080/collector'
    echo 'set httpd port 2812 and use address localhost'
    echo '  allow localhost'
    echo '  allow '$USER:$PASSWORD
} > /etc/monit/monitrc

monit reload
monit validate
```
