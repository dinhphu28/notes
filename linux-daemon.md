# Daemon

## Daemon Template

> Daemon need filename as <daemon-template>@.service
>
> E.g: filename is ssh-tunnel-client@.service
>
> Parameter is after @
>
> When using template:
>
> ```sh
> sudo systemctl start ssh-tunnel-client@web_01
> sudo systemctl status ssh-tunnel-client@web_01
> sudo systemctl enable ssh-tunnel-client@web_01
> ```

##### Daemon template for ssh tunneling (port forwarding)

###### ssh-tunnel-client@.service

```properties
[Unit]
Description=SSH Tunnel Service to %I
After=network.target

[Service]
EnvironmentFile=/etc/ssh_tunnel_client/%i
ExecStart=/usr/bin/ssh -v -F /etc/default/secure-tunnel.config -N -T -R 0.0.0.0:${REMOTE_PORT}:localhost:${LOCAL_PORT} ${USER}@${HOST}
Restart=on-failure
RestartSec=10
StandardOutput=file:/var/log/ssh_tunnel_client/%i.log

[Install]
WantedBy=default.target
```

###### secure-tunnel.config

This is .ssh/config file but specified by command in ExecStart with `id_nopass` is ssh private key (without passphrase)

```properties
Host your.server.com
     Hostname your.server.com
     User youruser
     IdentityFile /etc/ssh_priv/id_nopass
     ServerAliveInterval 60
     ExitOnForwardFailure yes
```

###### EnvironmentFile

This filename is parameter of daemon tempate, e.g: `web_01` (path: `/etc/ssh_tunnel_client/web_01`)

```properties
HOST=your.server.com
LOCAL_PORT=443
REMOTE_PORT=8443
USER=youruser
```

###### Log

Log will save to /var/log/ssh_tunnel_client/<parameter>.log

E.g: /var/log/ssh_tunnel_client/web_01.log
