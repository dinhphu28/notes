# SSH

#### Add a user for ssh-ing without shell permission

```sh
sudo adduser myuser01 --shell=/bin/false --no-create-home
```

Because of homeless user, we need to specify the `authorized-keys` file to ssh with PubkeyAuthentication

Append this to `/etc/ssh/sshd_config`

```conf
Match User phund30
        AuthorizedKeysFile /etc/ssh/authorized-keys/%u
```

Then

```sh
sudo mkdir -p /etc/ssh/authorized-keys/myuser01
sudo rmdir /etc/ssh/authorized-keys/myuser01
touch /etc/ssh/authorized-keys/myuser01
sudo touch /etc/ssh/authorized-keys/myuser01
sudo nvim /etc/ssh/authorized-keys/myuser01
sudo systemctl restart ssh
```

