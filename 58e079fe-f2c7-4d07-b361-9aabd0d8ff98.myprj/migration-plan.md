# MIGRATION FROM CHEF TO ANSIBLE

## Executive Summary

This repository contains a Chef-based infrastructure for deploying a multi-site Nginx web server with SSL, caching services (Memcached and Redis), and a FastAPI Python application with PostgreSQL. The migration to Ansible is estimated to be of moderate complexity, requiring approximately 2-3 weeks of effort for a skilled Ansible developer.

The repository consists of three main Chef cookbooks with clear responsibilities and minimal interdependencies, making it suitable for an incremental migration approach. The migration will involve converting Chef recipes, templates, and attributes to Ansible roles, tasks, templates, and variables while maintaining the same functionality and security posture.

## Module Migration Plan

This repository contains Chef cookbooks that need individual migration planning:

### MODULE INVENTORY

- **nginx-multisite**:
    - Description: Configures Nginx web server with multiple SSL-enabled virtual hosts, security hardening, and firewall configuration
    - Path: cookbooks/nginx-multisite
    - Technology: Chef
    - Key Features: Multi-site configuration, SSL certificate generation, fail2ban integration, UFW firewall configuration

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

- `Berksfile`: Defines cookbook dependencies including nginx (~> 12.0), memcached (~> 6.0), and redisio (~> 7.2.4)
- `Policyfile.rb`: Defines the run list and cookbook dependencies
- `solo.json`: Contains configuration data for the Nginx sites and security settings
- `solo.rb`: Chef Solo configuration file
- `Vagrantfile`: Defines a Fedora 42 VM for development and testing
- `vagrant-provision.sh`: Shell script to provision the Vagrant VM with Chef

### Target Details

- **Operating System**: Fedora/RHEL-based (Fedora 42 specified in Vagrantfile) with support for Ubuntu 18.04+ and CentOS 7+ (from cookbook metadata)
- **Virtual Machine Technology**: Libvirt (specified in Vagrantfile)
- **Cloud Platform**: Not specified, appears to be designed for on-premises deployment

## Migration Approach

### Key Dependencies to Address

- **nginx (~> 12.0)**: Replace with Ansible Galaxy role `geerlingguy.nginx` or create a custom Nginx role
- **memcached (~> 6.0)**: Replace with Ansible Galaxy role `geerlingguy.memcached`
- **redisio (~> 7.2.4)**: Replace with Ansible Galaxy role `geerlingguy.redis` or DavidWittman.redis
- **ssl_certificate (~> 2.1)**: Replace with Ansible's `openssl_*` modules for certificate generation

### Security Considerations

- **SSL Certificate Management**: The current implementation generates self-signed certificates. Migration should maintain this functionality using Ansible's `openssl_certificate` module.
- **Firewall Configuration**: UFW configuration should be migrated to use Ansible's `ufw` module or `firewalld` module depending on the target OS.
- **fail2ban Integration**: Configuration should be migrated using Ansible's template module for fail2ban configuration.
- **SSH Hardening**: SSH configuration hardening should be maintained using Ansible's template module or dedicated SSH role.
- **Redis Authentication**: Redis password authentication must be preserved in the migration.

### Technical Challenges

- **Multi-site Nginx Configuration**: The dynamic generation of Nginx site configurations based on node attributes will need to be replicated using Ansible's template module and variables.
- **Redis Configuration Hack**: The current implementation includes a Ruby block to modify Redis configuration files. This will need to be handled using Ansible's lineinfile module or template module with proper conditionals.
- **PostgreSQL User/Database Creation**: The current implementation uses a bash script executed via Chef. This should be replaced with Ansible's postgresql_user and postgresql_db modules.

### Migration Order

1. **nginx-multisite** (moderate complexity, foundation for web services)
   - Start with basic Nginx installation and configuration
   - Add SSL certificate generation
   - Add security hardening (fail2ban, UFW)
   - Add multi-site configuration

2. **cache** (moderate complexity, dependent services)
   - Implement Memcached configuration
   - Implement Redis with authentication

3. **fastapi-tutorial** (high complexity, application deployment)
   - Implement PostgreSQL installation and configuration
   - Implement Python environment setup
   - Implement application deployment from Git
   - Implement systemd service configuration

### Assumptions

1. The target environment will continue to be Fedora/RHEL-based systems, with possible deployment to Ubuntu/Debian systems.
2. Self-signed certificates are acceptable for development/testing purposes.
3. The current security posture (fail2ban, UFW, SSH hardening) should be maintained.
4. The FastAPI application repository at https://github.com/dibanez/fastapi_tutorial.git will remain available.
5. The Redis configuration hack is necessary due to compatibility issues with the Redis version.
6. The current directory structure in the target environment (/opt/server/*, /etc/ssl/*) should be maintained.
7. The migration will not involve changes to the application code or database schema.