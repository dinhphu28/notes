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

Then add ssh pubkey to file `/etc/ssh/authorized-keys/myuser01`

Restart sshd daemon

```sh
sudo systemctl restart ssh
```

