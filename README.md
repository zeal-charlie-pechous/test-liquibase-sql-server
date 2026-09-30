# test-liquibase-sql-server

Minimal standalone Maven + Liquibase project targeting Microsoft SQL Server.

## Contents

- `docker-compose.yml` – local SQL Server 2022 instance
- `pom.xml` – `liquibase-maven-plugin` with a patched SQL Server JDBC driver (`mssql-jdbc` 12.10.2.jre11)
- `src/main/resources/db/changelog/db.changelog-master.xml` – master changelog
- `src/main/resources/db/changelog/changes/001-create-person-table.xml` – sample change set creating a `person` table
- `.env.example` – template for credentials (the real `.env` is git-ignored)

## Requirements

- JDK 11+
- Maven 3.8+
- Docker with Compose v2

## Setup

1. Create your local environment file and set strong values:

   ```bash
   cp .env.example .env
   ```

   `.env` is listed in `.gitignore`, so credentials stay out of version control.

2. Load the variables into your shell (Docker Compose and the Maven build both read them from the environment):

   ```bash
   set -a && source .env && set +a
   ```

3. Start SQL Server:

   ```bash
   docker compose up -d
   ```

4. Create the target database once the container is healthy:

   ```bash
   docker compose exec sqlserver /opt/mssql-tools18/bin/sqlcmd \
     -S localhost -U sa -P "$MSSQL_SA_PASSWORD" -C \
     -Q "IF DB_ID('liquibase_demo') IS NULL CREATE DATABASE liquibase_demo"
   ```

## Migration commands

```bash
# Show pending change sets
mvn liquibase:status

# Apply migrations
mvn liquibase:update

# Preview the SQL without applying it
mvn liquibase:updateSQL

# Roll back the last change set
mvn liquibase:rollback -Dliquibase.rollbackCount=1

# Show applied change sets
mvn liquibase:history
```

## Teardown

```bash
docker compose down -v
```

## Notes

- Connection settings are supplied only through the environment variables `LIQUIBASE_URL`, `LIQUIBASE_USERNAME` and `LIQUIBASE_PASSWORD`; no credentials are stored in the repository.
- The JDBC URL in `.env.example` uses `encrypt=true;trustServerCertificate=true`, which is appropriate for the local self-signed container certificate only. For non-local environments use `trustServerCertificate=false` and a trusted certificate.
- `mssql-jdbc` is pinned to a patched release; keep it updated when new security fixes are published.
