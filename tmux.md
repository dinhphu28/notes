# tmux

## Create new session with new socket

This is usefull when you want to start a new tmux socket with new config, ignore the default `~/.tmux.conf` file.

```sh
tmux -L socket-name -f /dev/null new -s session-name
```
