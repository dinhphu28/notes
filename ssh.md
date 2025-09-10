# SSH

## Add a user for ssh-ing without shell permission

```sh
sudo adduser phund30 --shell=/bin/false --no-create-home
```

Because of homeless user, we need to specify the `authorized-keys` file to ssh with PubkeyAuthentication

Append this to `/etc/ssh/sshd_config`

```conf
Match User phund30
        AuthorizedKeysFile /etc/ssh/authorized-keys/%u
```

Then add ssh pubkey to file `/etc/ssh/authorized-keys/phund30`

Restart sshd daemon

```sh
sudo systemctl restart ssh
```

## Setup SSH Tunneling (Remote) in Linux Systemd

Before, enable GatewayPorts in `/etc/ssh/sshd_config`

```properties
GatewayPorts yes
```

or

```properties
GatewayPorts clientspecified
```

Let get started

> Using user `phund30` has been created before.

### Create config

Create ssh config file `/etc/default/secure-tunnel.config`

```structured text
Host node01.dinhphu28.com
     Hostname node01.dinhphu28.com
     User phund30
     IdentityFile /etc/ssh_priv/id_nopass
     ServerAliveInterval 60
     ExitOnForwardFailure yes
```

### Create systemd template

Create systemd file `/etc/systemd/system/ssh-tunnel-client@.service`

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

Create your env to use with template

```sh
# Create env directory if not exist
mkdir /etc/ssh_tunnel_client
```

**E.g.** You want to create profile name: `http_proxy_7082`

Create file `/etc/ssh_tunnel_client/http_proxy_7082`

```properties
HOST=your-host.dinhphu28.com
LOCAL_PORT=7082
REMOTE_PORT=7082
USER=phund30
```

### Start your service

```sh
sudo systemctl start ssh-tunnel-client@http_proxy_7082
```

To enable it to start automatically at boot

```sh
sudo systemctl enable ssh-tunnel-client@http_proxy_7082
```

