# Migration Plan: fastapi-tutorial

**TLDR**: This cookbook installs and configures a FastAPI Python web application with PostgreSQL database backend. It sets up a single FastAPI instance running on port 8000, with a dedicated PostgreSQL database, Python virtual environment, and systemd service.

## Service Type and Instances

**Service Type**: Application Server (Python FastAPI web application)

**Configured Instances**:
- **fastapi-tutorial**: Python FastAPI web application
  - Location/Path: /opt/fastapi-tutorial
  - Port/Socket: 8000
  - Key Config: 
    - Database: PostgreSQL (fastapi_db)
    - Database User: fastapi
    - Environment Variables: PROJECT_NAME, API_VERSION, DATABASE_URL

## File Structure

```
cookbooks/fastapi-tutorial/recipes/default.rb
cookbooks/fastapi-tutorial/metadata.rb
```

## Module Explanation

The cookbook performs operations in this order:

1. **default** (`cookbooks/fastapi-tutorial/recipes/default.rb`):
   - Installs Python 3 and required system packages: python3, python3-pip, python3-venv, git, postgresql, postgresql-contrib, libpq-dev
   - Creates application directory: /opt/fastapi-tutorial
   - Clones FastAPI tutorial repository from GitHub (https://github.com/dibanez/fastapi_tutorial.git, branch: main)
   - Creates Python virtual environment at /opt/fastapi-tutorial/venv
   - Installs Python dependencies from requirements.txt using pip
   - Enables and starts PostgreSQL service
   - Creates PostgreSQL database (fastapi_db) and user (fastapi) with password
   - Creates environment configuration file (.env) with project settings
   - Creates systemd service file for the FastAPI application
   - Reloads systemd configuration when service file changes
   - Enables and starts the FastAPI service
   - Resources: package (1), directory (1), git (1), execute (3), service (2), file (2)

## Dependencies

**External cookbook dependencies**: None specified in metadata.rb
**System package dependencies**: python3, python3-pip, python3-venv, git, postgresql, postgresql-contrib, libpq-dev
**Service dependencies**: postgresql.service (required for FastAPI service)

## Checks for the Migration

**Files to verify**:
- /opt/fastapi-tutorial/ (application directory)
- /opt/fastapi-tutorial/venv/ (Python virtual environment)
- /opt/fastapi-tutorial/.env (environment configuration)
- /etc/systemd/system/fastapi-tutorial.service (systemd service file)

**Service endpoints to check**:
- Ports listening: 8000 (FastAPI web application)
- PostgreSQL: 5432 (default PostgreSQL port)

**Templates rendered**:
- /opt/fastapi-tutorial/.env (environment configuration file)
- /etc/systemd/system/fastapi-tutorial.service (systemd service file)

## Pre-flight checks:
```bash
# Service status
systemctl status fastapi-tutorial
systemctl status postgresql
ps aux | grep uvicorn

# Application health
curl -I http://localhost:8000/
curl -s http://localhost:8000/docs  # FastAPI auto-generated docs

# Database connectivity
sudo -u postgres psql -c "\l" | grep fastapi_db
sudo -u postgres psql -c "\du" | grep fastapi
PGPASSWORD=fastapi_password psql -h localhost -U fastapi -d fastapi_db -c "SELECT 1;"

# Configuration validation
cat /opt/fastapi-tutorial/.env | grep -E 'PROJECT_NAME|API_VERSION|DATABASE_URL'
cat /etc/systemd/system/fastapi-tutorial.service
systemctl show fastapi-tutorial

# Python environment
ls -la /opt/fastapi-tutorial/venv/bin/
/opt/fastapi-tutorial/venv/bin/python --version
/opt/fastapi-tutorial/venv/bin/pip list | grep -E 'fastapi|uvicorn|sqlalchemy'

# Git repository
cd /opt/fastapi-tutorial && git remote -v
cd /opt/fastapi-tutorial && git branch

# Logs
journalctl -u fastapi-tutorial -f
journalctl -u postgresql -f

# Network listening
netstat -tulpn | grep 8000
ss -tlnp | grep 8000
lsof -i :8000

# PostgreSQL listening
netstat -tulpn | grep 5432
ss -tlnp | grep postgres
```