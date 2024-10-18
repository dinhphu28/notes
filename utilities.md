# Utilities

### Password generation with Linux

Because Linux have `/dev/random` & `/dev/urandom`, we can use them to generate ramdom password.

###### Printable characters excluding space:

```sh
< /dev/urandom tr -dc '[:graph:]' | head -c32; echo
```

###### Printables characters, including space:

```sh
< /dev/urandom tr -dc '[:print:]' | head -c32; echo
```

###### All letters:

```sh
< /dev/urandom tr -dc '[:alnum:]' | head -c32; echo
```

> With -c32 is the length of the password
>
> We can use `/dev/random` for stronger security

For more, read man page of `tr`

### Unlock Desktop Session via CLI (SSH)

> Note that I've just done on openSUSE Tumbleweed with KDE Plasma 6 (Wayland)

```sh
loginctl list-sessions
loginctl unlock-session 3 # With 3 is your desktop session, it can be another
loginctl lock-session 3
```

