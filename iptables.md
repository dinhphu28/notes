# IPTABLES

### Filter

```sh
sudo iptables -I FORWARD 1 -s 10.8.0.22 -j DROP
sudo iptables -A FORWARD -s 10.8.0.22 -d 10.8.0.1 -p tcp --dport 7082 -j ACCEPT
```

- Drop all traffic from 10.8.0.22 (with `I` and `1` will put this rule on top)
- Forward traffic from 10.8.0.22 to 10.8.0.1 via TCP/7082

```sh
sudo iptables -A INPUT -i eth0 -m state --state NEW -p udp --dport 1194 -j ACCEPT
```

Allow packets from `eth0` with state NEW via UDP/1194

```sh
sudo iptables -A OUTPUT -o tun+ -j ACCEPT
sudo iptables -A INPUT -i tun+ -j ACCEPT
sudo iptables -A FORWARD -i tun+ -j ACCEPT
```

- Allow packets leaving the system through `tun+`
- Allow packets entering the system from `tun+`
- Allow forwarding traffic comes from `tun+` to other interfaces

```sh
sudo iptables -A FORWARD -i tun+ -o eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i eth0 -o tun+ -m state --state RELATED,ESTABLISHED -j ACCEPT
```

Forwarding traffic from `tun+` to `eth0` and from `eth0` to `tun+` match the connection state which packets are part of an existing connection (ESTABLISHED) or related to and existing connection (RELATED)

```sh
sudo iptables -A FORWARD -i eth0 -o tun+ -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i tun+ -o eth0 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
```

These command equivalent the previous commands, but the `conntrack` module is modern than `state` and has better performance, more feature-rich and future-proof

### NAT

```sh
# sudo iptables -t nat -A POSTROUTING -o tun0 -j MASQUERADE # Don't remember whether it needed! But now it isn't needed!
```

```sh
sudo iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
```

Setup NAT for subnet 10.8.0.0/24 going through the tun interface

#### TCP

```sh
sudo iptables -A FORWARD -i eth0 -o tun+ -p tcp --syn --dport 443 -m conntrack --ctstate NEW -j ACCEPT
sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 443 -j DNAT --to-destination 10.8.0.14
sudo iptables -t nat -A POSTROUTING -o tun+ -p tcp --dport 443 -d 10.8.0.14 -j SNAT --to-source 10.8.0.1
```

Port forwarding traffic from `eth0` to 10.8.0.14 at `tun+` (443:443) with protocol TCP

#### UDP

```sh
sudo iptables -A FORWARD -i eth0 -o tun+ -p udp --dport 443 -m conntrack --ctstate NEW -j ACCEPT
sudo iptables -t nat -A PREROUTING -i eth0 -p udp --dport 443 -j DNAT --to-destination 10.8.0.14
sudo iptables -t nat -A POSTROUTING -o tun+ -p udp --dport 443 -d 10.8.0.14 -j SNAT --to-source 10.8.0.1
```

Port forwarding traffic from `eth0` to 10.8.0.14 at `tun+` (443:443) with protocol UDP

##### To save:

```sh
sudo netfilter-persistent save
```

Rules will be saved at `/etc/iptables/rules.v4`