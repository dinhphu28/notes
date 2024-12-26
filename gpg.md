# GPG

### GPG Agent

Config file: `~/.gnupg/gpg-agent.conf`

Example:

```properties
default-cache-ttl 600
pinentry-program /usr/bin/pinentry-curses
no-allow-external-cache
```

With `pinentry-curses` is TUI Dialog for GPG Agent.

To these changes taked effect, run below commands:

```sh
gpgconf --kill gpg-agent
gpg-agent --daemon --use-standard-socket
gpg-connect-agent reloadagent /bye
```

In case agent not response

```sh
gpg-agent --daemon --use-standard-socket
```

