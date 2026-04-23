# sql-server-docker-demo

A `docker-compose` setup that spins up a Microsoft SQL Server container with a **persistent volume** so your data survives container restarts and re-creations.

## Prerequisites

### 1. Install WSL2

WSL2 (Windows Subsystem for Linux 2) is required as the backend for Podman on Windows.

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

### 2. Install Podman Desktop for Windows

Podman runs natively on Windows and uses WSL2 as its container backend — no Docker Desktop required.

1. Download the latest **Podman Desktop** installer from [podman-desktop.io](https://podman-desktop.io/).

2. Run the installer. When prompted, allow it to install the **Podman CLI** and initialize a **Podman machine** backed by WSL2.

3. After installation, open a new PowerShell or Command Prompt window and verify:

   ```powershell
   podman --version
   podman machine list
   ```

   The machine should be in the **Running** state. If it is not, start it:

   ```powershell
   podman machine start
   ```

4. Install **podman-compose** (the Compose-compatible front-end):

   ```powershell
   pip install podman-compose
   ```

   Verify the installation:

   ```powershell
   podman-compose --version
   ```

## Quick start

All commands below are run from a **Windows PowerShell or Command Prompt** window from the root of this repository.

1. **Copy the example environment file and set your SA password:**

   ```powershell
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

SQL Server stores its data files under `/var/opt/mssql` inside the container. The `docker-compose.yml` maps that directory to a named Podman volume (`sqlserver-data`), which is managed by Podman (via the WSL2 backend) on your Windows machine. The data persists as long as the volume exists — even if you remove and recreate the container.

## Configuration

All tuneable settings live in `.env` (see `.env.example`):

| Variable     | Default       | Description |
|--------------|---------------|-------------|
| `SA_PASSWORD` | *(required)* | SA account password |
| `MSSQL_PID`  | `Developer`   | SQL Server edition |
| `HOST_PORT`  | `1433`        | Host port mapped to the container |