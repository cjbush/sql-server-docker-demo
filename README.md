# sql-server-docker-demo

A `docker-compose` setup that spins up a Microsoft SQL Server container with a **persistent volume** so your data survives container restarts and re-creations.

## Prerequisites

### 1. Install WSL2

WSL2 (Windows Subsystem for Linux 2) is required to run Linux containers on Windows.

1. Open **PowerShell as Administrator** and run:

   ```powershell
   wsl --install
   ```

   This installs WSL2 and the default Ubuntu distribution. Reboot when prompted.

2. After rebooting, open the **Ubuntu** app from the Start menu and complete the initial Linux user setup (username + password).

3. Verify WSL2 is active:

   ```powershell
   wsl --list --verbose
   ```

   The Ubuntu entry should show `VERSION 2`. If it shows `1`, upgrade it:

   ```powershell
   wsl --set-version Ubuntu 2
   ```

> **Note:** WSL2 requires Windows 10 version 2004 (build 19041) or later, or Windows 11. Run `winver` to check your build.

### 2. Install Podman inside WSL2

All commands below are run inside the **Ubuntu WSL2 terminal**.

1. Update packages and install Podman:

   ```bash
   sudo apt-get update
   sudo apt-get install -y podman
   ```

2. Verify the installation:

   ```bash
   podman --version
   ```

3. Install **podman-compose** (the Compose-compatible front-end):

   ```bash
   sudo apt-get install -y python3-pip
   pip3 install podman-compose
   ```

   Or, if `pipx` is available:

   ```bash
   pipx install podman-compose
   ```

4. Verify podman-compose:

   ```bash
   podman-compose --version
   ```

## Quick start

All commands below are run inside the **Ubuntu WSL2 terminal** from the root of this repository.

1. **Copy the example environment file and set your SA password:**

   ```bash
   cp .env.example .env
   ```

   Open `.env` and replace the placeholder values — at minimum set `SA_PASSWORD` to a strong password that meets SQL Server's complexity requirements (≥ 8 characters, mixed case, digit, special character).

2. **Start the container:**

   ```bash
   podman-compose up -d
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
   podman exec -it sql-server /opt/mssql-tools18/bin/sqlcmd \
     -S localhost -U sa -P '<YourPassword>' -No
   ```

4. **Stop the container (data is preserved):**

   ```bash
   podman-compose down
   ```

5. **Stop the container _and_ remove all data:**

   ```bash
   podman-compose down -v
   ```

## How persistence works

SQL Server stores its data files under `/var/opt/mssql` inside the container. The `docker-compose.yml` maps that directory to a named Podman volume (`sqlserver-data`), which is managed by Podman on your local machine (inside WSL2). The data persists as long as the volume exists — even if you remove and recreate the container.

## Configuration

All tuneable settings live in `.env` (see `.env.example`):

| Variable     | Default       | Description |
|--------------|---------------|-------------|
| `SA_PASSWORD` | *(required)* | SA account password |
| `MSSQL_PID`  | `Developer`   | SQL Server edition |
| `HOST_PORT`  | `1433`        | Host port mapped to the container |