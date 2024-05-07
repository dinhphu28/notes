# ZSH

### Edit current command with text editor (e.g. neovim)

##### Editing

Use `Ctrl + X` then `Ctrl + E`

##### Set editor to neovim

```sh
export EDITOR="nvim"
export VISUAL="nvim"
```

You can add below lines to `~/.zshrc`

### Using packages installed by BREW with root priviliges (sudo)

Open file `/etc/sudoers`

If not found this file, can use command (I remember that):

```sh
sudo vim sudoers
```

Then find this line

```conf
Defaults secure_path="<something here>"
```

Then add `:/home/linuxbrew/.linuxbrew/bin:/home/linuxbrew/.linuxbrew/sbin/`

Result look like this:

```conf
Defaults secure_path="/usr/sbin:/usr/bin:/sbin:/bin:/home/linuxbrew/.linuxbrew/bin:/home/linuxbrew/.linuxbrew/sbin/"
```

### Read from user input (e.g. password, ...)

```sh
read -r "username?Enter username: " && \
read -rs "password?Enter password: "
```

Result:

```sh
echo $username
echo $password
```

> Remember to `unset password` for security
