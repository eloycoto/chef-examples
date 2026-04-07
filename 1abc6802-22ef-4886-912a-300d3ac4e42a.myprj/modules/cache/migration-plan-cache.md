# Migration Plan: cache

**TLDR**: This cookbook configures two caching services: Memcached and Redis. It sets up a single Memcached instance with default configuration and a single Redis instance on port 6379 with password authentication. The cookbook handles installation, configuration, and service management for both services.

## Service Type and Instances

**Service Type**: Cache

**Configured Instances**:

- **memcached**: Default Memcached instance
  - Location/Path: /etc/memcached.conf
  - Port/Socket: 11211 (TCP and UDP)
  - Key Config: 64MB memory, 1024 max connections

- **redis-6379**: Redis instance with authentication
  - Location/Path: /etc/redis/6379.conf
  - Port/Socket: 6379
  - Key Config: Password authentication enabled, default memory settings

## File Structure

**Recipes:**
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
```

**Providers:**
```
/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/providers/configure.rb
/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/providers/install.rb
```

**Templates:**
```
/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/templates/default/redis.conf.erb
/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/templates/default/redis.init.erb
/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/templates/default/redis.upstart.conf.erb
/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/templates/default/redis@.service.erb
```

**Attributes:**
```
/workspace/source/migration-dependencies/cookbook_artifacts/memcached-7992788f1a376defb902059063f5295e37d281cb/attributes/default.rb
/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/attributes/default.rb
```

## Module Explanation

The cookbook performs operations in this order:

1. **cache::default** (`cookbooks/cache/recipes/default.rb`):
   - Includes memcached recipe
   - Sets Redis server configuration with password authentication
   - Creates Redis log directory
   - Includes Redis recipes
   - Applies configuration fixes via ruby_block
   - Resources: include_recipe (3), directory (1), ruby_block (1)

2. **memcached::default** (`/workspace/source/migration-dependencies/cookbook_artifacts/memcached-7992788f1a376defb902059063f5295e37d281cb/recipes/default.rb`):
   - Uses custom resource memcached_instance['memcached'] to configure a single Memcached instance
   - Sets memory (64MB), port (11211), max connections (1024), and other parameters
   - Resources: memcached_instance (1)

3. **redisio::default** (`/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/recipes/default.rb`):
   - Updates apt repositories
   - Includes prerequisite installation recipes
   - Includes Redis installation, OS default service disabling, and configuration recipes
   - Resources: apt_update (1), include_recipe (3), build_essential (1)

4. **redisio::_install_prereqs** (`/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/recipes/_install_prereqs.rb`):
   - Installs required packages for Redis
   - Resources: package (multiple, based on platform)

5. **redisio::install** (`/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/recipes/install.rb`):
   - Installs Redis either from package or source based on configuration
   - If using package: installs redis-server package
   - If using source: builds Redis from source
   - Includes ulimit configuration
   - Resources: package (1) or redisio_install (1), build_essential (1)

6. **redisio::ulimit** (`/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/recipes/ulimit.rb`):
   - Configures system ulimit settings for Redis
   - On Debian systems: configures PAM settings
   - Resources: template (1), cookbook_file (1), user_ulimit (conditional)

7. **redisio::disable_os_default** (`/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/recipes/disable_os_default.rb`):
   - Stops and disables the default OS Redis service
   - Resources: service (1)

8. **redisio::configure** (`/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/recipes/configure.rb`):
   - Uses custom resource redisio_configure['redis-servers'] to configure Redis instances
   - Creates service resources for each Redis instance based on job control system (systemd, initd, upstart)
   - Resources: redisio_configure (1), service (1 per instance)

9. **ruby_block[fix_redis_config]** (`cookbooks/cache/recipes/default.rb`):
   - Modifies the Redis configuration file to remove specific replication-related settings
   - Removes lines related to replica configuration
   - Resources: ruby_block (1)

10. **redisio::enable** (`/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/recipes/enable.rb`):
    - Enables and starts Redis services for each configured instance
    - For the single instance on port 6379
    - Resources: service (1)

## Dependencies

**External cookbook dependencies**:
- memcached (~> 6.0)
- redisio

**System package dependencies**:
- memcached
- redis-server (if using package installation)
- build-essential (if building Redis from source)

**Service dependencies**:
- memcached.service
- redis@6379.service (systemd) or redis6379 (initd/upstart)

## Checks for the Migration

**Files to verify**:
- /etc/memcached.conf
- /etc/redis/6379.conf
- /var/log/redis/
- /var/lib/redis/
- /etc/systemd/system/redis@6379.service (if using systemd)
- /etc/init.d/redis6379 (if using initd)
- /etc/init/redis6379.conf (if using upstart)

**Service endpoints to check**:
- Ports listening: 11211 (Memcached TCP/UDP), 6379 (Redis)
- Unix sockets: None explicitly configured
- Network interfaces: 0.0.0.0 (Memcached), Redis binds to default

**Templates rendered**:
- redis.conf.erb → /etc/redis/6379.conf (1 instance)
- redis@.service.erb → /etc/systemd/system/redis@6379.service (if using systemd)
- redis.init.erb → /etc/init.d/redis6379 (if using initd)
- redis.upstart.conf.erb → /etc/init/redis6379.conf (if using upstart)

## Pre-flight checks:
```bash
# Memcached checks
# Service status
systemctl status memcached
ps aux | grep memcached

# Connectivity
echo stats | nc localhost 11211
memcached-tool localhost:11211 stats
memcached-tool localhost:11211 display

# Configuration validation
cat /etc/memcached.conf
grep -E 'memory|port|listen|maxconn' /etc/memcached.conf

# Logs
journalctl -u memcached -f
tail -f /var/log/memcached.log

# Network listening
netstat -tulpn | grep 11211
ss -tulpn | grep 11211
lsof -i :11211

# Redis checks
# Service status
systemctl status redis@6379  # for systemd
service redis6379 status     # for initd
status redis6379            # for upstart
ps aux | grep redis

# Connectivity (with authentication)
redis-cli -p 6379 -a redis_secure_password_123 ping
redis-cli -p 6379 -a redis_secure_password_123 info

# Configuration validation
cat /etc/redis/6379.conf
grep -E 'port|requirepass|maxmemory|bind' /etc/redis/6379.conf

# Verify configuration fixes
grep -E 'replica-serve-stale-data|replica-read-only|repl-ping-replica-period|client-output-buffer-limit|replica-priority' /etc/redis/6379.conf
# These lines should be removed or commented out

# Logs
tail -f /var/log/redis/redis_6379.log
journalctl -u redis@6379 -f  # for systemd

# Network listening
netstat -tulpn | grep 6379
ss -tulpn | grep 6379
lsof -i :6379

# Data directories
ls -lah /var/lib/redis/
df -h /var/lib/redis/

# Memory usage
ps aux | grep redis | awk '{print $2}' | xargs -I{} cat /proc/{}/status | grep VmRSS
redis-cli -p 6379 -a redis_secure_password_123 info memory
```