## Wait for Postgres table to become available for query

`./wait-for-pgtable` is a script designed to synchronize services like docker containers. It is [sh](https://en.wikipedia.org/wiki/Bourne_shell) and [alpine](https://alpinelinux.org/) compatible.

It was inspired by [eficode/wait-for](https://github.com/eficode/wait-for), which was also inspired by [vishnubob/wait-for-it](https://github.com/vishnubob/wait-for-it).
I've forked and updated it to be used specifically for testing of Postgres connections.

`wait-for-pgtable` will perform next tests:
1. Try to connect to Postgres by PGHOST:PGPORT.
2. Try to query the TABLE (or several in a row). This will help to solve the case when Postgres is up but is not ready for querying yet.

### Requirements

When using this tool, you need:
1. Have `psql` command installed and available. i.e. for Alpine: `$ apk --no-cache add postgresql-client`
2. Define env variables for connection: PGHOST, PGPORT, PGDATABASE, PGUSER, PGPASSWORD
3. Pick the `wait-for-pgtable` file as part of your project.


## Usage

```
Usage:
  wait-for-pgtable TABLE [TABLE ...] [-t timeout] [-q] [-- command args]

  TABLE                               Name of the table to check for existence.
                                      Multiple table names are allowed - all
                                      of them will be checked.
  -q | --quiet                        Do not output status messages (only exit errors)
  -t TIMEOUT | --timeout=timeout      Timeout in seconds, zero for no timeout
  -- COMMAND ARGS                     Execute command with args after the test finishes

Connection to Postgres should be specified by environment variables:
PGHOST, PGPORT, PGDATABASE, PGUSER, PGPASSWORD.
More info: https://www.postgresql.org/docs/current/static/libpq-envars.html
```


## Examples

To check if table "MYTABLE" is available:

```
$ PGHOST=localhost \
  PGPORT=5432 \
  PGDATABASE=mydb \
  PGUSER=myuser \
  PGPASSWORD=mypassword \
  ./wait-for-pgtable MYTABLE  -- echo "Table is ready"

Table is ready
```

To wait for database container to become available:

```
version: '2'

services:
  db:
    image: postgres:9.4
    environment:
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: mypassword

  backend:
    build: backend
    environment:
      PGHOST: db
      PGPORT: 5432
      PGDATABASE: myuser
      PGUSER: myuser
      PGPASSWORD: mypassword
    command: ["./wait-for-pgtable", "MYTABLE", "-t", "30", "--", "/start_your_app"]
    depends_on:
      - db
```
