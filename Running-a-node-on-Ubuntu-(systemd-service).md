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
ExecStart=/bin/bash -c "/root/.cargo/bin/cargo run -- --ch$
```
