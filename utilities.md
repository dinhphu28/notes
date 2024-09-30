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