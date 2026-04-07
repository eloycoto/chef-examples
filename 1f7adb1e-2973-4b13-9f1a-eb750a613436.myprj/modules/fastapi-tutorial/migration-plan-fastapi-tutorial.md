# Migration Plan: fastapi-tutorial

**TLDR**: This cookbook deploys a FastAPI Python web application with a PostgreSQL database backend. It sets up a single FastAPI instance running on port 8000, configures a PostgreSQL database, and creates a systemd service to manage the application.

## Service Type and Instances

**Service Type**: Web Server (FastAPI Python Application)

**Configured Instances**:

- **fastapi-tutorial**: Python FastAPI web application
  - Location/Path: /opt/fastapi-tutorial
  - Port/Socket: 8000
  - Key Config: 
    - Database URL: postgresql://fastapi:fastapi_password@localhost/fastapi_db
    - Project Name: "FastAPI Tutorial"
    - API Version: 1.0.0

- **postgresql**: PostgreSQL database server
  - Location/Path: Default PostgreSQL installation paths
  - Port/Socket: 5432 (default PostgreSQL port)
  - Key Config:
    - Database: fastapi_db
    - User: fastapi
    - Password: fastapi_password

## File Structure

```
cookbooks/fastapi-tutorial/recipes/default.rb
cookbooks/fastapi-tutorial/metadata.rb
```

## Module Explanation

The cookbook performs operations in this order:

1. **default** (`cookbooks/fastapi-tutorial/recipes/default.rb`):
   - Installs required system packages: python3, python3-pip, python3-venv, git, postgresql, postgresql-contrib, libpq-dev
   - Creates application directory: /opt/fastapi-tutorial
   - Clones FastAPI tutorial repository from https://github.com/dibanez/fastapi_tutorial.git (branch: main)
   - Creates Python virtual environment at /opt/fastapi-tutorial/venv
   - Installs Python dependencies from requirements.txt
   - Enables and starts PostgreSQL service
   - Creates PostgreSQL database (fastapi_db) and user (fastapi) with password
   - Creates environment configuration file (.env) with database connection details
   - Creates systemd service file for the FastAPI application
   - Enables and starts the FastAPI service
   - Resources: package (1), directory (1), git (1), execute (3), service (2), file (2)

## Dependencies

**External cookbook dependencies**: None specified in metadata.rb
**System package dependencies**: python3, python3-pip, python3-venv, git, postgresql, postgresql-contrib, libpq-dev
**Service dependencies**: postgresql.service (required by fastapi-tutorial.service)

## Checks for the Migration

**Files to verify**:
- /opt/fastapi-tutorial (application directory)
- /opt/fastapi-tutorial/venv (Python virtual environment)
- /opt/fastapi-tutorial/.env (environment configuration)
- /etc/systemd/system/fastapi-tutorial.service (systemd service file)

**Service endpoints to check**:
- Ports listening: 8000 (FastAPI application), 5432 (PostgreSQL)
- Network interfaces: 0.0.0.0 (FastAPI listens on all interfaces)

**Templates rendered**:
- No templates used, but inline content is written to:
  - /opt/fastapi-tutorial/.env
  - /etc/systemd/system/fastapi-tutorial.service

## Pre-flight checks:
```bash
# FastAPI service status
systemctl status fastapi-tutorial
ps aux | grep uvicorn

# PostgreSQL service status
systemctl status postgresql
ps aux | grep postgres

# Database connectivity
sudo -u postgres psql -c "SELECT 1;"
psql -U fastapi -d fastapi_db -h localhost -c "SELECT 1;"

# Application health
curl -I http://localhost:8000/
curl -s http://localhost:8000/docs | grep "FastAPI Tutorial"

# Environment configuration
cat /opt/fastapi-tutorial/.env
grep DATABASE_URL /opt/fastapi-tutorial/.env

# Python virtual environment
ls -la /opt/fastapi-tutorial/venv/bin/
/opt/fastapi-tutorial/venv/bin/python --version

# Service configuration
cat /etc/systemd/system/fastapi-tutorial.service
systemctl show fastapi-tutorial | grep ExecStart

# Network listening
netstat -tulpn | grep 8000
ss -tlnp | grep 8000
lsof -i :8000

# PostgreSQL listening
netstat -tulpn | grep 5432
ss -tlnp | grep postgres

# Logs
journalctl -u fastapi-tutorial -n 50
journalctl -u postgresql -n 50

# Repository check
ls -la /opt/fastapi-tutorial/.git
cd /opt/fastapi-tutorial && git remote -v

# Application dependencies
/opt/fastapi-tutorial/venv/bin/pip list
cat /opt/fastapi-tutorial/requirements.txt
```