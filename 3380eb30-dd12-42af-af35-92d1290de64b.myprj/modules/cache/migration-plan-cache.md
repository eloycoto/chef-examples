# Migration Plan: cache

**TLDR**: This cookbook installs and configures two caching services: Memcached and Redis. It sets up a single Memcached instance on port 11211 and a single Redis instance on port 6379 with password authentication. The Redis configuration includes a custom fix to remove specific replication-related configuration lines.

## Service Type and Instances

**Service Type**: Cache

**Configured Instances**:

- **memcached**: Standard Memcached instance
  - Location/Path: System default (/var/lib/memcached)
  - Port/Socket: 11211
  - Key Config: 64MB memory, 1024 max connections

- **redis-6379**: Redis instance with authentication
  - Location/Path: /var/lib/redis
  - Port/Socket: 6379
  - Key Config: Password authentication enabled with 'redis_secure_password_123'

## File Structure

```
cookbooks/cache/recipes/default.rb
/workspace/source/migration-dependencies/cookbook_artifacts/memcached-7992788f1a376defb902059063f5295e37d281cb/recipes/default.rb
/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/recipes/default.rb
/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/recipes/_install_prereqs.rb
/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/recipes/install.rb
/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/recipes/ulimit.rb
/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/recipes/disable_os_default.rb
/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/recipes/configure.rb
/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/recipes/enable.rb
/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/providers/configure.rb
/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/providers/install.rb
/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/templates/default/redis.conf.erb
/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/templates/default/redis.init.erb
/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/templates/default/redis@.service.erb
/workspace/source/migration-dependencies/cookbook_artifacts/memcached-7992788f1a376defb902059063f5295e37d281cb/attributes/default.rb
/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/attributes/default.rb
```

## Module Explanation

The cookbook performs operations in this order:

1. **default** (`cookbooks/cache/recipes/default.rb`):
   - Includes memcached recipe
   - Sets Redis server configuration with authentication
   - Creates Redis log directory
   - Includes Redis recipes
   - Fixes Redis configuration with a ruby_block
   - Resources: include_recipe (3), directory (1), ruby_block (1)

2. **memcached::default** (`/workspace/source/migration-dependencies/cookbook_artifacts/memcached-7992788f1a376defb902059063f5295e37d281cb/recipes/default.rb`):
   - Uses custom resource: memcached_instance['memcached']
   - Configures Memcached with 64MB memory, port 11211, max connections 1024
   - Resources: memcached_instance (1)

3. **redisio::default** (`/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/recipes/default.rb`):
   - Updates apt repositories
   - Includes prerequisite installation recipes
   - Includes Redis installation recipes
   - Resources: apt_update (1), include_recipe (3)

4. **redisio::_install_prereqs** (`/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/recipes/_install_prereqs.rb`):
   - Installs prerequisite packages based on platform
   - Resources: package (multiple based on platform)

5. **redisio::install** (`/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/recipes/install.rb`):
   - Installs Redis from source or package based on configuration
   - Uses custom resource: redisio_install['redis-installation']
   - Includes ulimit recipe
   - Resources: build_essential (1), redisio_install (1), include_recipe (1)

6. **redisio::ulimit** (`/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/recipes/ulimit.rb`):
   - Configures ulimit settings for Redis
   - Sets up PAM configuration for ulimit on Debian platforms
   - Resources: template (1), cookbook_file (1), user_ulimit (conditional)

7. **redisio::disable_os_default** (`/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/recipes/disable_os_default.rb`):
   - Disables default OS Redis service if present
   - Resources: service (1) with stop and disable actions

8. **redisio::configure** (`/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/recipes/configure.rb`):
   - Uses custom resource: redisio_configure['redis-servers']
   - Creates Redis configuration files
   - Sets up service files based on init system (systemd, initd, upstart, or rcinit)
   - Resources: redisio_configure (1), service (1 per Redis instance)
   - Provider: /workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/providers/configure.rb
     - Creates Redis user and group
     - Creates configuration directories
     - Creates data directories
     - Creates PID directories
     - Creates log directories
     - Renders Redis configuration template
     - Sets up init scripts based on job control system
     - Templates:
       - If systemd: redis@.service.erb → /lib/systemd/system/redis@6379.service
       - If initd: redis.init.erb → /etc/init.d/redis6379
       - If upstart: redis.upstart.conf.erb → /etc/init/redis6379.conf
       - If rcinit: redis.rcinit.erb → /usr/local/etc/rc.d/redis6379

9. **ruby_block[fix_redis_config]** (`cookbooks/cache/recipes/default.rb`):
   - Modifies Redis configuration file at /etc/redis/6379.conf
   - Removes specific replication-related configuration lines:
     - replica-serve-stale-data
     - replica-read-only
     - repl-ping-replica-period
     - client-output-buffer-limit
     - replica-priority
   - Resources: ruby_block (1)

10. **redisio::enable** (`/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/recipes/enable.rb`):
    - Enables and starts Redis service
    - Service name depends on job control system:
      - If systemd: redis@6379
      - Otherwise: redis6379
    - Resources: service (1) with start and enable actions

## Dependencies

**External cookbook dependencies**:
- memcached
- redisio

**System package dependencies**:
- memcached
- build-essential (for Redis compilation if not using package)
- Redis dependencies (varies by platform)

**Service dependencies**:
- memcached.service
- redis@6379.service or redis6379 (depending on init system)

## Checks for the Migration

**Files to verify**:
- /etc/memcached.conf
- /etc/redis/6379.conf
- /var/log/redis/
- /var/lib/redis/
- /var/run/redis/
- /lib/systemd/system/redis@6379.service (if using systemd)
- /etc/init.d/redis6379 (if using initd)
- /etc/init/redis6379.conf (if using upstart)

**Service endpoints to check**:
- Ports listening: 11211 (Memcached), 6379 (Redis)
- Unix sockets: None explicitly configured

**Templates rendered**:
- Redis configuration template: 1 instance (for port 6379)
- Redis service template: 1 instance (for port 6379)

## Pre-flight checks:
```bash
# Memcached checks
systemctl status memcached
ps aux | grep memcached
netstat -tulpn | grep 11211
ss -tlnp | grep memcached
echo "stats" | nc localhost 11211
echo "version" | nc localhost 11211

# Redis checks
# Service status
systemctl status redis@6379 || service redis6379 status
ps aux | grep redis-server

# Redis connectivity
redis-cli -h localhost -p 6379 -a redis_secure_password_123 PING
redis-cli -h localhost -p 6379 -a redis_secure_password_123 INFO server
redis-cli -h localhost -p 6379 -a redis_secure_password_123 INFO clients
redis-cli -h localhost -p 6379 -a redis_secure_password_123 INFO memory

# Configuration validation
cat /etc/redis/6379.conf | grep -E 'port|requirepass'
cat /etc/redis/6379.conf | grep -v 'replica-serve-stale-data'
cat /etc/redis/6379.conf | grep -v 'replica-read-only'
cat /etc/redis/6379.conf | grep -v 'repl-ping-replica-period'
cat /etc/redis/6379.conf | grep -v 'client-output-buffer-limit'
cat /etc/redis/6379.conf | grep -v 'replica-priority'

# Logs
tail -f /var/log/redis/redis-server.log
journalctl -u redis@6379 -f

# Network listening
netstat -tulpn | grep 6379
ss -tlnp | grep redis
lsof -i :6379

# Data directories
ls -lah /var/lib/redis/
ls -lah /var/run/redis/
df -h /var/lib/redis/

# Memory usage
ps aux | grep redis-server | awk '{print $2}' | xargs -I {} cat /proc/{}/status | grep VmRSS
```