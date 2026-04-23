# sql-server-docker-demo

A `docker-compose` setup that spins up a Microsoft SQL Server container with a **persistent volume** so your data survives container restarts and re-creations.

## Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (or Docker Engine + Compose plugin on Linux)

## Quick start

1. **Copy the example environment file and set your SA password:**

   ```bash
   cp .env.example .env
   ```

   Open `.env` and replace the placeholder values — at minimum set `SA_PASSWORD` to a strong password that meets SQL Server's complexity requirements (≥ 8 characters, mixed case, digit, special character).

2. **Start the container:**

   ```bash
   docker compose up -d
   ```

3. **Connect to SQL Server:**

   | Setting  | Value         |
   |----------|---------------|
   | Host     | `localhost`   |
   | Port     | `1433` (or whatever you set `HOST_PORT` to) |
   | Username | `sa`          |
   | Password | value of `SA_PASSWORD` in your `.env` |

   Using `sqlcmd` (bundled inside the container):

   ```bash
   docker exec -it sql-server /opt/mssql-tools18/bin/sqlcmd \
     -S localhost -U sa -P '<YourPassword>' -No
   ```

4. **Stop the container (data is preserved):**

   ```bash
   docker compose down
   ```

5. **Stop the container _and_ remove all data:**

   ```bash
   docker compose down -v
   ```

## How persistence works

SQL Server stores its data files under `/var/opt/mssql` inside the container. The `docker-compose.yml` maps that directory to a named Docker volume (`sqlserver-data`), which is managed by Docker on your local machine. The data persists as long as the volume exists — even if you remove and recreate the container.

## Configuration

All tuneable settings live in `.env` (see `.env.example`):

| Variable     | Default       | Description |
|--------------|---------------|-------------|
| `SA_PASSWORD` | *(required)* | SA account password |
| `MSSQL_PID`  | `Developer`   | SQL Server edition |
| `HOST_PORT`  | `1433`        | Host port mapped to the container |