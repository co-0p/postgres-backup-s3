# Introduction
This project provides Docker images to periodically back up a PostgreSQL database to AWS S3, and to restore from the backup as needed.

> This is a fork of [eeshugerman's](https://github.com/eeshugerman/postgres-backup-s3), and then [alexanderbartels'](https://github.com/alexanderbartels/postgres-backup-s3)
> version of **postgres-backup-s3**, simply updated to support Postgres18.
>  You can see the first fork [dif here](https://github.com/eeshugerman/postgres-backup-s3/compare/master...alexanderbartels:postgres-backup-s3:master),
> and the second fork [dif here](https://github.com/alexanderbartels/postgres-backup-s3/compare/master...co-0p:postgres-backup-s3:master).

# Usage
## Backup
```yaml
version: '3.8'
services:
  postgres:
    image: postgres:18
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
  # Remember to tune your shared buffers when using any dockerized postgres

  pg_backup:
    image: c0op/postgres-backup-s3:18
    environment:
      SCHEDULE: '@weekly'     # optional
      BACKUP_KEEP_DAYS: 7     # optional
      PASSPHRASE: passphrase  # optional
      S3_REGION: region
      S3_ACCESS_KEY_ID: key
      S3_SECRET_ACCESS_KEY: secret
      S3_BUCKET: my-bucket
      S3_PREFIX: backup
      POSTGRES_HOST: postgres
      POSTGRES_DATABASE: dbname
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
```

- Images are tagged by the major PostgreSQL version supported: `12`, `13`, `14`, `15`, `16`, `17`, or `18`.
- The `SCHEDULE` variable determines backup frequency. See go-cron schedules documentation [here](http://godoc.org/github.com/robfig/cron#hdr-Predefined_schedules). Omit to run the backup immediately and then exit.
- If `PASSPHRASE` is provided, the backup will be encrypted using GPG.
- Run `docker exec <container name> sh backup.sh` to trigger a backup ad-hoc.
- If `BACKUP_KEEP_DAYS` is set, backups older than this many days will be deleted from S3.
- Set `S3_ENDPOINT` if you're using a non-AWS S3-compatible storage provider.

## Restore
> [!CAUTION]
> DATA LOSS POTENTIAL! All database objects will be dropped and re-created.

### ... from latest backup
```sh
docker exec <container name> sh restore.sh
```

> [!NOTE]
> If your bucket has more than a 1000 files, the latest may not be restored -- only one S3 `ls` command is used

### ... from specific backup
```sh
docker exec <container name> sh restore.sh <timestamp>
```

# Development
## Build the image locally
`ALPINE_VERSION` determines Postgres version compatibility. See [`build-and-push-images.yml`](.github/workflows/build-and-push-images.yml) for the latest mapping.
```sh
DOCKER_BUILDKIT=1 docker build --build-arg ALPINE_VERSION=3.14 .
```
## Run a simple test environment with Docker Compose
```sh
cp template.env .env
# fill out your secrets/params in .env
docker compose up -d
```

# Acknowledgements
This project is a fork and re-structuring of @schickling's, then @eeshugerman's, then @alexanderbartels' projects.

