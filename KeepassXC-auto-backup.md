# KeepassXC auto backup

`/home/dinhphu28/.local/bin/keepass_backup.sh`

```sh
#!/bin/bash

DB_NAME="dinhphu28-vault"
DB_PATH="$HOME/OneDrive-dinhphu28/no_any_secret/${DB_NAME}.kdbx"
BACKUP_DIR="$HOME/no_any_secret/KeePassBackups/"
MAX_BACKUPS=10

# Create backup directory if it doesn't exist
mkdir -p "$BACKUP_DIR"

# Timestamp
TIMESTAMP=$(date +"%Y%m%d_%H%M%S")

# Backup file
BACKUP_FILE="$BACKUP_DIR/${DB_NAME}_$TIMESTAMP.kdbx"

# Copy the database
cp "$DB_PATH" "$BACKUP_FILE"

# Remove old backups if exceeding MAX_BACKUPS
BACKUPS_COUNT=$(ls -1 "$BACKUP_DIR"/${DB_NAME}_*.kdbx 2>/dev/null | wc -l)
if [ "$BACKUPS_COUNT" -gt "$MAX_BACKUPS" ]; then
    # Delete oldest backups
    ls -1t "$BACKUP_DIR"/${DB_NAME}_*.kdbx | tail -n +$((MAX_BACKUPS+1)) | xargs -r rm --
fi
```

`~/.config/systemd/user/keepass-backup.service`

```sh
[Unit]
Description=KeePassXC backup

[Service]
Type=oneshot
ExecStart=/home/dinhphu28/.local/bin/keepass_backup.sh
```

`~/.config/systemd/user/keepass-backup.timer`

```sh
[Unit]
Description=Run KeePassXC backup every day

[Timer]
OnCalendar=daily
; OnCalendar=hourly
; Every minute
; OnCalendar=*:0/1:00
Persistent=true

[Install]
WantedBy=timers.target
```

Then run

```sh
systemctl --user enable --now keepass-backup.timer
```

---

> [!Important]
>
> User timers run only when: user systemd session is active.
> So it doesn't run if system boots and you never log in.

If your want it to run even when not logged in => Enable **linger**

```sh
loginctl enable-linger $USER
```

