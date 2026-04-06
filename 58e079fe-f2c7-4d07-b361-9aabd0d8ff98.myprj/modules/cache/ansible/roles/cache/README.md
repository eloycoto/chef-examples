# Cache Role

This Ansible role installs and configures caching services:
- Memcached
- Redis

## Requirements

- Ansible 2.9 or higher
- Required collections:
  - community.general
  - ansible.posix

## Role Variables

### Memcached Variables

```yaml
# Memcached user
memcached_user: memcache

# Memcached configuration
memcached_port: 11211
memcached_listen_address: 127.0.0.1
memcached_max_connections: 1024
memcached_memory: 64
memcached_enable_sasl: false

# Memcached file paths
memcached_conf_file: "/etc/memcached.conf" # Debian
memcached_log_file: "/var/log/memcached.log"
```

### Redis Variables

```yaml
# Redis server configuration
redis_servers:
  - port: "6379"
    requirepass: "redis_secure_password_123"

# Redis user and group
redis_user: redis
redis_group: redis

# Redis directories
redis_conf_dir: /etc/redis
redis_data_dir: /var/lib/redis
redis_log_dir: /var/log/redis

# Redis service name format
redis_service_name: "redis@{{ redis_servers[0].port }}"
```

## Example Playbook

```yaml
- hosts: cache_servers
  roles:
    - role: cache
      vars:
        memcached_memory: 128
        redis_servers:
          - port: "6379"
            requirepass: "my_secure_password"
```

## License

Apache-2.0

## Author Information

Chef Example