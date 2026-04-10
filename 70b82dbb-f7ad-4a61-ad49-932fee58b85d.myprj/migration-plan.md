# MIGRATION FROM CHEF TO ANSIBLE

## Executive Summary

This repository contains a Chef-based infrastructure setup for a multi-site Nginx web server with caching services (Redis and Memcached) and a FastAPI application backed by PostgreSQL. The migration to Ansible will involve converting three Chef cookbooks, handling external dependencies, and ensuring proper security configurations are maintained.

**Estimated Timeline:**
- Analysis and Planning: 1 week
- Development of Ansible roles: 3-4 weeks
- Testing and Validation: 2 weeks
- Documentation and Knowledge Transfer: 1 week
- Total: 7-8 weeks

**Complexity Assessment:** Medium to High
- Multiple interconnected services
- Security configurations that need careful migration
- Database and application deployment requirements

## Module Migration Plan

This repository contains Chef cookbooks that need individual migration planning:

### MODULE INVENTORY

- **nginx-multisite**:
    - Description: Configures Nginx with multiple SSL-enabled virtual hosts, security hardening, and site configurations
    - Path: cookbooks/nginx-multisite
    - Technology: Chef
    - Key Features: Multi-site configuration, SSL certificate generation, security hardening (fail2ban, ufw)

- **cache**:
    - Description: Configures caching services including Memcached and Redis with authentication
    - Path: cookbooks/cache
    - Technology: Chef
    - Key Features: Redis with password authentication, Memcached configuration

- **fastapi-tutorial**:
    - Description: Deploys a FastAPI Python application with PostgreSQL database backend
    - Path: cookbooks/fastapi-tutorial
    - Technology: Chef
    - Key Features: Python virtual environment setup, Git repository deployment, PostgreSQL database configuration, systemd service management

### Infrastructure Files

- `Berksfile`: Defines cookbook dependencies (nginx, memcached, redisio, ssl_certificate)
- `Policyfile.rb`: Defines the Chef policy with run list and cookbook dependencies
- `solo.json`: Contains node configuration data including Nginx site configurations and security settings
- `solo.rb`: Chef Solo configuration file
- `Vagrantfile`: Defines a Fedora 42 VM for development/testing with port forwarding
- `vagrant-provision.sh`: Bash script to provision the Vagrant VM with Chef

### Target Details

- **Operating System**: Fedora (based on Vagrantfile specifying "generic/fedora42")
- **Virtual Machine Technology**: Vagrant with libvirt provider
- **Cloud Platform**: Not specified, appears to be designed for local development/testing

## Migration Approach

### Key Dependencies to Address

- **nginx (~> 12.0)**: Replace with Ansible Galaxy role `geerlingguy.nginx` or create a custom Nginx role
- **memcached (~> 6.0)**: Replace with Ansible Galaxy role `geerlingguy.memcached`
- **redisio (~> 7.2.4)**: Replace with Ansible Galaxy role `geerlingguy.redis` or DavidWittman.redis
- **ssl_certificate (~> 2.1)**: Replace with Ansible's `openssl_*` modules for certificate management

### Security Considerations

- **fail2ban configuration**: Migrate using Ansible's package and template modules to maintain the same configuration
- **ufw firewall rules**: Use Ansible's `ufw` module to configure the same firewall rules
- **SSH hardening**: Use Ansible's `lineinfile` or templates to configure SSH security settings
- **Redis authentication**: Ensure Redis password is stored securely in Ansible Vault
- **PostgreSQL credentials**: Store database credentials in Ansible Vault
- **SSL certificates**: Ensure proper handling of SSL certificates and private keys with appropriate permissions

### Technical Challenges

- **Multi-site Nginx configuration**: Ensure the Ansible role can handle multiple virtual hosts with SSL as flexibly as the Chef cookbook
- **Service dependencies**: Maintain proper ordering of service deployments (database before application, etc.)
- **Idempotency**: Ensure all operations are idempotent, particularly database user/schema creation
- **Environment variables**: Properly handle the `.env` file for the FastAPI application
- **Custom Nginx configurations**: Ensure security.conf and other custom configurations are properly migrated

### Migration Order

1. **nginx-multisite role** (foundation for web services)
   - Base Nginx installation and configuration
   - SSL certificate generation
   - Virtual host configuration
   - Security hardening (fail2ban, ufw)

2. **cache role** (supporting services)
   - Memcached configuration
   - Redis installation and security configuration

3. **fastapi-tutorial role** (application layer)
   - PostgreSQL installation and configuration
   - Python environment setup
   - Application deployment
   - Service configuration

### Assumptions

1. The target environment will continue to be Fedora-based systems
2. The same security requirements will apply in the new environment
3. Self-signed certificates are acceptable for development (production would likely use Let's Encrypt or similar)
4. The FastAPI application repository at https://github.com/dibanez/fastapi_tutorial.git will remain available
5. The same Redis and PostgreSQL passwords can be used (should be replaced with Ansible Vault variables)
6. The current Nginx site configuration structure will be maintained

## Ansible Structure Recommendation

```
ansible-nginx-multisite/
├── inventories/
│   ├── development/
│   │   ├── group_vars/
│   │   │   └── all/
│   │   │       ├── vars.yml
│   │   │       └── vault.yml (encrypted)
│   │   └── hosts.yml
│   └── production/
├── roles/
│   ├── nginx_multisite/
│   │   ├── defaults/
│   │   ├── handlers/
│   │   ├── tasks/
│   │   └── templates/
│   ├── cache_services/
│   │   ├── defaults/
│   │   ├── handlers/
│   │   ├── tasks/
│   │   └── templates/
│   └── fastapi_app/
│       ├── defaults/
│       ├── handlers/
│       ├── tasks/
│       └── templates/
├── playbooks/
│   ├── site.yml
│   ├── nginx.yml
│   ├── cache.yml
│   └── fastapi.yml
├── requirements.yml (for Ansible Galaxy dependencies)
└── vagrant.yml (for local testing)
```

## Testing Strategy

1. Create a Vagrant environment similar to the current one but using Ansible provisioning
2. Develop and test each role independently
3. Integrate roles and test the complete stack
4. Verify all security configurations are properly applied
5. Test idempotency by running the playbooks multiple times
6. Validate application functionality through the Nginx proxy