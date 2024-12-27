# Utilities

### Password generation

#### Linux

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

#### macOS

With macOS, an error occur when run below command because of UTF-8. To fix it, add LC_ALL=C as below:

```sh
< /dev/urandom LC_ALL=C tr -dc '[:print:]' | head -c32; echo
```

##### Other tools

```sh
openssl rand -base64 32
```

```sh
date | md5
```

```sh
gpg --gen-random 1 32
```

### Unlock Desktop Session via CLI (SSH)

> Note that I've just done on openSUSE Tumbleweed with KDE Plasma 6 (Wayland & X11)

```sh
loginctl list-sessions
loginctl unlock-session 3 # With 3 is your desktop session, it can be another
loginctl lock-session 3
```

### Hide content in an image

#### With `steghide`

```sh
steghide embed -cf image.jpg -ef secret.txt
steghide extract -sf image.jpg
```

#### With `cat`

```sh
cat image.jpg secret.txt > combined.jpg
tail -c <size_of_secret.txt> combined.jpg > recovered.txt
```

#### With `xxd`

```sh
xxd -p image.jpg > image.hex
xxd -p secret.txt > secret.hex
cat image.hex secret.hex > combined.hex
xxd -r -p combined.hex combined.jpg
```

#### OR can use GUI `OpenStego`

https://github.com/syvaidya/openstego

### Sync file to cloud

Using `rclone`

E.g:

```sh
rclone copy "$LOCAL_PATH" "$REMOTE_DIR" --progress
```

### Create random bytes file

```sh
dd if=/dev/urandom of=random.key bs=99K count=3
```

