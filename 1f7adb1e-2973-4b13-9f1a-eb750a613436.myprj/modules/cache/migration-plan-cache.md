# Migration Plan: Cache

**TLDR**: This cookbook sets up a caching infrastructure with both Memcached and Redis services. It configures a single Memcached instance with default settings and one Redis instance on port 6379 with password authentication. The cookbook handles installation, configuration, and service management for both caching solutions.

## Service Type and Instances

**Service Type**: Cache

**Configured Instances**:

- **memcached**: Default Memcached instance
  - Location/Path: System default
  - Port/Socket: 11211 (TCP and UDP)
  - Key Config: 64MB memory, 1024 max connections, listening on 0.0.0.0

- **redis-6379**: Redis instance
  - Location/Path: /etc/redis/6379.conf
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
/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/templates/default/redis.upstart.conf.erb
/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/templates/default/redis@.service.erb
/workspace/source/migration-dependencies/cookbook_artifacts/memcached-7992788f1a376defb902059063f5295e37d281cb/attributes/default.rb
/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/attributes/default.rb
```

## Module Explanation

The cookbook performs operations in this order:

1. **default** (`cookbooks/cache/recipes/default.rb`):
   - Includes memcached recipe
   - Sets Redis server configuration with password authentication
   - Creates Redis log directory
   - Includes Redis recipes
   - Fixes Redis configuration with a ruby_block
   - Resources: include_recipe (3), directory (1), ruby_block (1)

2. **memcached::default** (`/workspace/source/migration-dependencies/cookbook_artifacts/memcached-7992788f1a376defb902059063f5295e37d281cb/recipes/default.rb`):
   - Uses custom resource memcached_instance to configure a single Memcached instance
   - Sets memory to 64MB, port to 11211, max connections to 1024
   - Resources: memcached_instance (1)

3. **redisio::default** (`/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/recipes/default.rb`):
   - Updates apt packages if needed
   - Includes prerequisite installation recipes
   - Includes Redis installation recipe
   - Resources: apt_update (1), include_recipe (2)

4. **redisio::_install_prereqs** (`/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/recipes/_install_prereqs.rb`):
   - Installs prerequisite packages for Redis
   - Resources: package (multiple, one for each prerequisite)

5. **redisio::install** (`/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/recipes/install.rb`):
   - Installs Redis either from package or source based on configuration
   - If using package: installs redis-server package
   - If using source: builds Redis from source
   - Includes ulimit recipe for system limits configuration
   - Resources: package (1) or redisio_install (1), build_essential (1)

6. **redisio::ulimit** (`/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/recipes/ulimit.rb`):
   - Configures system ulimit settings for Redis
   - On Debian systems, configures PAM settings
   - Resources: template (1), cookbook_file (1), user_ulimit (conditional)

7. **redisio::disable_os_default** (`/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/recipes/disable_os_default.rb`):
   - Disables default OS Redis service if present
   - Resources: service (1) with stop and disable actions

8. **redisio::configure** (`/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/recipes/configure.rb`):
   - Configures Redis instances based on the servers attribute
   - Uses redisio_configure custom resource to set up Redis configuration
   - Creates service resources based on job control system (systemd, initd, upstart, or rcinit)
   - Resources: redisio_configure (1), service (1 per Redis instance)

9. **ruby_block[fix_redis_config]** (`cookbooks/cache/recipes/default.rb`):
   - Modifies the Redis configuration file to remove specific lines
   - Removes replica-related configuration options
   - Resources: ruby_block (1)

10. **redisio::enable** (`/workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/recipes/enable.rb`):
    - Enables and starts Redis services for each configured instance
    - For the single instance on port 6379
    - Resources: service (1) with start and enable actions

## Dependencies

**External cookbook dependencies**:
- memcached
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
- /etc/redis/6379.conf
- /var/log/redis/
- /var/lib/redis/
- /etc/memcached.conf (or distribution-specific location)

**Service endpoints to check**:
- Ports listening:
  - 11211 (Memcached TCP and UDP)
  - 6379 (Redis)
- Unix sockets: None explicitly configured
- Network interfaces: 
  - Memcached: 0.0.0.0 (all interfaces)
  - Redis: Default (typically 127.0.0.1)

**Templates rendered**:
- Redis configuration template (redis.conf.erb) - rendered once for port 6379
- Redis service template (based on init system) - rendered once

## Pre-flight checks:
```bash
# Memcached checks
systemctl status memcached
ps aux | grep memcached
netstat -tulpn | grep 11211
ss -tulpn | grep 11211
memcached-tool 127.0.0.1:11211 stats
echo "stats" | nc localhost 11211
echo "version" | nc localhost 11211

# Redis checks
systemctl status redis@6379  # For systemd
service redis6379 status     # For initd
ps aux | grep redis
netstat -tulpn | grep 6379
ss -tulpn | grep 6379

# Redis connectivity test with authentication
redis-cli -h localhost -p 6379 -a 'redis_secure_password_123' ping
redis-cli -h localhost -p 6379 -a 'redis_secure_password_123' info server
redis-cli -h localhost -p 6379 -a 'redis_secure_password_123' info memory

# Redis configuration check
grep -v '^#' /etc/redis/6379.conf | grep -v '^$'
grep "requirepass" /etc/redis/6379.conf
cat /etc/redis/6379.conf | grep -E 'port|bind|requirepass'

# Check for removed configuration lines
! grep "replica-serve-stale-data" /etc/redis/6379.conf
! grep "replica-read-only" /etc/redis/6379.conf
! grep "repl-ping-replica-period" /etc/redis/6379.conf
! grep "client-output-buffer-limit" /etc/redis/6379.conf
! grep "replica-priority" /etc/redis/6379.conf

# Check log files
tail -f /var/log/redis/redis_6379.log
tail -f /var/log/memcached.log

# Check system limits
cat /proc/$(pgrep -f "redis-server.*:6379")/limits
cat /proc/$(pgrep memcached)/limits

# Check data directories
ls -la /var/lib/redis/
ls -la /var/lib/memcached/ # if applicable
```