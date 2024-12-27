# Rotating Logs With Logrotate in Linux

With **openSUSE Tumbleweed** logrotate has been installed by default, else install it.

## Setup logrotate for a service

1. Create a logrotate configuration file in `/etc/logrotate.d/`

​	Example: `http_proxy_8072_fp` service

2. Write your config

```properties
/var/log/http_proxy_7082_fp/*.log {
    compress
    dateext
    dateformat -%Y-%m-%d
    daily
    rotate 7
    missingok
    notifempty
    delaycompress
    copytruncate
}
```

Explain:

- Auto rotate the `*.log` files
- Compress
- Use date extension with format `yyyy/mm/dd`. E.g: `access.log-2024-12-21` or `access.log-2024-12-21.xz`
- Rotate daily
- Only keep 7 day
- If the log filke is missing, go on to the next one without issuing an error message
- Do not rotate the log if it is empty
- Delay compress to the next rotation cycle, this only has effect when used in combination with `compress`
- Truncate the original log file to zero size in place after creating a copy, instead of moving the old log file and optionally creating a new one. It can be used when some program cannot be told to close its logfile and thus might continue writing (appending) to previous log file forever. Note that there is a very small time slice between copying the file and truncating it, so some logging data might be lost.

3. Verifying

```sh
sudo logrotate -d /etc/logrotate.d/http_proxy_7082_fp
```

4. Apply

```sh
sudo logrotate /etc/logrotate.d/http_proxy_7082_fp
```

If we want logrotate to evaluate the condition immediately, we can pass the -f option to force run it:

```sh
sudo logrotate -f /etc/logrotate.d/http_proxy_7082_fp
```

