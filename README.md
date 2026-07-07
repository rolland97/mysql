# MySQL + phpMyAdmin

Local MySQL 8.4 database with phpMyAdmin web UI, run via Docker Compose.

## Prerequisites

- Docker
- Docker Compose (v2, `docker compose`)

## Setup

Copy the example env file and edit values:

```bash
cp .env.example .env
```

| Variable | Default | Description |
|---|---|---|
| `MYSQL_ROOT_PASSWORD` | `root-password` | Root password |
| `MYSQL_DATABASE` | `ewp` | Database created on first start |
| `MYSQL_USER` | `ewp` | Non-root user |
| `MYSQL_PASSWORD` | `ewp-password` | Password for `MYSQL_USER` |
| `FORWARD_DB_PORT` | `3307` | Host port → container `3306` |
| `FORWARD_PHPMYADMIN_PORT` | `8080` | Host port → phpMyAdmin |

> `.env` is gitignored. Never commit real passwords.

## Up

Start the stack (detached):

```bash
docker compose up -d
```

- MySQL: `localhost:3307`
- phpMyAdmin: <http://localhost:8080>

Check status / logs:

```bash
docker compose ps
docker compose logs -f
```

## Down

Stop and remove containers (keeps data):

```bash
docker compose down
```

Stop and remove containers **and delete database data**:

```bash
docker compose down -v
```

> `-v` drops the `db_data` volume. All database contents lost. Use only for a clean reset.

## Update `.env`

Edit `.env`, then recreate containers to apply:

```bash
docker compose up -d --force-recreate
```

**What changes and what doesn't:**

| Variable | Applies on recreate? |
|---|---|
| `FORWARD_DB_PORT` | ✅ Yes |
| `FORWARD_PHPMYADMIN_PORT` | ✅ Yes |
| `MYSQL_ROOT_PASSWORD` | ⚠️ No — only on first init |
| `MYSQL_DATABASE` | ⚠️ No — only on first init |
| `MYSQL_USER` | ⚠️ No — only on first init |
| `MYSQL_PASSWORD` | ⚠️ No — only on first init |

> `MYSQL_*` creds are read **only when the `db_data` volume is empty** (first
> start). Editing them later does nothing to the existing database. To apply
> new creds you must either change them inside MySQL by hand, or do a fresh
> env (below).

## Fresh env (reset from scratch)

Wipes all data and re-initializes MySQL with current `.env` values:

```bash
docker compose down -v
docker compose up -d
```

`down -v` deletes the `db_data` volume → next `up` runs first-time init and
applies every `MYSQL_*` value from `.env`.

> **Destructive.** All databases, tables, and rows are permanently lost. Back
> up first if needed:
>
> ```bash
> docker compose exec db mysqldump -u root -p --all-databases > backup.sql
> ```

## Connect

```bash
mysql -h 127.0.0.1 -P 3307 -u ewp -p
```

Data persists across `up`/`down` in the `db_data` volume (unless you pass `-v`).
