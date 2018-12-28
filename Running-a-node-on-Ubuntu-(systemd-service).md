To create a systemd service:
```
systemctl edit --force --full edgeware.service
```

- --force causes a new service to be created
- --full ensures the service is created as a full unit file, and not a snippet

```
[Unit]
Description=Edgeware Node daemon
[Service]
Type=exec
WorkingDirectory=/root/edgeware-node
ExecStart=/bin/bash -c "/root/.cargo/bin/cargo run -- --chain=edgeware --key <key> --name <name>"
[Install]
WantedBy=multi-user.target
```

Change WorkingDirectory and ExecStart to their appropriate values (`which cargo` to find where cargo is installed). WantedBy specifies when the service starts.

```
systemctl start edgeware
```

To start the service on system startup:
```
systemctl enable edgeware
```
