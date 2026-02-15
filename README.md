# pg-backup

Small wrapper around `pg_dump` and `pg_restore` for parallel PostgreSQL dump/restore.

## Requirements

- `bash`
- PostgreSQL client tools (`pg_dump`, `pg_restore`)
- Access to the target database (`PGUSER`, `PGPASSWORD`, `PGHOST`, etc. as needed)

## Install with mise

Install the released tool from GitHub Releases:

```bash
mise use --global github:parisni/pg-backup@v1.0.0
```

Or pin in a project-local `mise.toml`:

```toml
[tools]
"github:parisni/pg-backup" = "v1.0.0"
```

Then run:

```bash
mise install
```

### If `mise` says "No matching asset found"

`mise` can cache release metadata. If the release assets were uploaded after the tag was first seen, clear cache and retry:

```bash
mise cache clear -y
mise use --global github:parisni/pg-backup@v1.0.0
```

## Usage

```bash
pg-backup dump|restore --worker N --db DBNAME --dir DIRECTORY
```

## Connection Setup (Important)

`pg-backup` uses PostgreSQL client defaults and environment variables for connection settings.

Set these before running:

- `PGHOST`: PostgreSQL host (example: `127.0.0.1` or `db.internal`)
- `PGPORT`: PostgreSQL port (example: `5432`)
- `PGUSER`: PostgreSQL user
- `PGPASSWORD`: PostgreSQL password (optional if you use `.pgpass`)

Database name is still passed with `--db`.

### Recommended: use `.pgpass` for password

Create `~/.pgpass`:

```text
hostname:port:database:username:password
127.0.0.1:5432:mydb:app:secret
```

Restrict permissions (required by PostgreSQL):

```bash
chmod 600 ~/.pgpass
```

Then run without `PGPASSWORD`:

```bash
PGHOST=127.0.0.1 PGPORT=5432 PGUSER=app \
pg-backup dump --worker 8 --db mydb --dir ./backup_dir
```

### Quick setup with environment variables

```bash
export PGHOST=127.0.0.1
export PGPORT=5432
export PGUSER=app
export PGPASSWORD='secret'
```

Run:

```bash
pg-backup dump --worker 8 --db mydb --dir ./backup_dir
pg-backup restore --worker 8 --db mydb --dir ./backup_dir
```

Arguments:

- `dump|restore` (required): action to run
- `--worker N` (optional, default `4`): number of parallel jobs used by `pg_dump`/`pg_restore`
- `--db DBNAME` (required): database name
- `--dir DIRECTORY` (required): directory for the dump output (for `dump`) or input (for `restore`)

Examples:

```bash
# Create a parallel directory-format backup
PGUSER=postgres pg-backup dump --worker 8 --db app --dir ./backup_dir

# Restore from that backup
PGUSER=postgres pg-backup restore --worker 8 --db app --dir ./backup_dir
```

The script calls:

- `dump`: `pg_dump -F d -j <workers> -Z 6 -f <dir>`
- `restore`: `pg_restore -j <workers> --clean --if-exists <dir>`
