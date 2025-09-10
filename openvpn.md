# OpenVPN3 Client for Linux

### Connection guide

> Note that you need to config SELinux if used

```sh
sudo openvpn3 config-import --config /path/to/file.ovpn --name my-vpn
sudo openvpn3 config-manage --config my-vpn --allow-compression yes
sudo openvpn3 configs-list
sudo openvpn3 session-start --config my-vpn
```

### Configuration for each client (by .ovpn profile)

#### Add Client Config Directory

Add this line to `/etc/openvpn/server.conf`

> client-config-dir ccd

Create file in ccd (`/etc/openvpn/ccd`) with filename the same as CN (Common Name) of Client profile (e.g: client_01)

E.g: `/etc/openvpn/ccd/client_01` content:

> ifconfig-push 10.8.0.22 10.8.0.21
>
> push "redirect-gateway def1 bypass-dhcp"

The first line to assign static IP address 10.8.0.22 via gateway 10.8.0.21 to client_01. (For more information about gateway 10.8.0.21 but not 10.8.0.1, google "subnet 32 of OpenVPN")

#### Push specific route for Client profile

Add these line to client_01.ovpn file:

> route 10.8.0.0 255.255.255.0
> 
> route 192.168.169.0 255.255.255.0
> 
> pull-filter ignore "redirect-gateway"
> 
> pull-filter ignore "dhcp-option DNS"

- Add route to 10.8.0.0 & 192.168.169.0
- Ignore push default "redirect-gateway"
- Ignore push DNS

> dhcp-option PROXY_HTTP 10.8.0.1 7082
> 
> dhcp-option PROXY_HTTPS 10.8.0.1 7082

Setup HTTP Proxy when connect.

