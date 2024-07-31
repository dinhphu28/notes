# OpenVPN3 Client for Linux

### Connection guide

> Note that you need to config SELinux if used

```sh
sudo openvpn3 config-import --config /path/to/file.ovpn --name my-vpn
sudo openvpn3 config-manage --config my-vpn --allow-compression yes
sudo openvpn3 configs-list
sudo openvpn3 session-start --config my-vpn
```