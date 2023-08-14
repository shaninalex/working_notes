---
title: "Simple Database Backup"
date: 2023-08-15T00:30:48+03:00
draft: false
tags: ['database']
---

Recently I was needed to make daily backup of database. This is always a good idea to have most reacent backup of
data or source codes. But daily backups... Sound like a perfect task for automation!

> Most of a time I'm working with PostgreSQL database. And example implemets backups for same db.

The bash script:

```bash
#!/bin/bash

date=$(date '+%Y-%m-%d')

PGPASSWORD="database_password" \
pg_dump \
--host 127.0.0.1 \
--port 5432 \
-U database_user \
--verbose \
--file "backups/DB_backup_$date.sql" \
database_name
```

Flags:
- `host` database host
- `port` database port
- `U` username
- `verbose`  show detailed information
- `file` filepath to store backup 


And crontab string:

```bash
0 1 * * * ./postgres_db_backup_daily.sh
```

And this is it. Simple backups. You can change file name pattern if you want... I think of extending this script and send saved file to email...
But this is for next time.


## Restore

```bash
$ psql -U username -d dbname -1 -f filename.sql
```

