# Firefly III Backup

#### firefly-iii_maria_db backup

Use to backup (dump) only database, to backup `upload` files, use docker volume backup as next section.

> [!TIP]
>
> With this, we can backup ***without stop*** the database, safe for ***cross version*** compatible.

```sh
set -a
source $HOME/firefly-iii/.db.env
set +a

export VERSION="$(date +%F)"
docker exec firefly_iii_db \
  /usr/bin/mariadb-dump -u"$MYSQL_USER" -p"$MYSQL_PASSWORD" "$MYSQL_DATABASE" \
  | gzip > $HOME/backups/firefly/firefly-$VERSION.sql
```

#### firefly-iii docker volume backup

> [!IMPORTANT]
>
> Because of backup raw docker volume, ***stop*** the containers ***before*** do the backup.

```sh
docker compose stop

export VERSION="$(date +%F)"
docker run --rm -v "firefly-iii_firefly_iii_db:/tmp" \
    -v "$HOME/backups/firefly:/backup:Z" \
    alpine tar -czvf /backup/firefly-iii_firefly_db-$VERSION.tar /tmp

docker run --rm -v "firefly-iii_firefly_iii_upload:/tmp" \
    -v "$HOME/backups/firefly:/backup:Z" \
    alpine tar -czvf /backup/firefly-iii_firefly_upload-$VERSION.tar /tmp
```

#### docker volume restore

Use to restore the database volume backup as above.

```sh
export VERSION='2026-03-27'
docker run --rm -v "firefly-iiii_firefly_iii_db:/recover" \
    -v "$HOME/backups/firefly:/backup:Z" \
    apline tar -xvf /backup/firefly-iii_firefly_db-$VERSION.tar -C /recover --strip 1
```

Then run docker compose up to start the new firefly-iii with backed up data.

---

#### Run with mariadb standalone container

Use to create a standalone database from backup.

```sh
docker run -d --name mariadb_tmp \
	-p 3316:3306 \
	-v "firefly_iii_db:/var/lib/mysql:Z" \
	--env-file .db.env \
	mariadb:lts
```

