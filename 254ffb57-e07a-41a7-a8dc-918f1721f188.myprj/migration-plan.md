# MIGRATION FROM CHEF TO ANSIBLE

## Executive Summary

This repository contains a Chef-based infrastructure for deploying a multi-site Nginx web server with SSL, caching services (Redis and Memcached), and a FastAPI Python application with PostgreSQL. The migration to Ansible is estimated to be of medium complexity, requiring approximately 3-4 weeks of effort for a single engineer or 1-2 weeks with a team of 2-3 engineers.

The repository consists of three main Chef cookbooks with clear responsibilities and minimal cross-dependencies, making this a good candidate for incremental migration. The primary challenge will be preserving the security configurations and SSL certificate management during the transition.

## Module Migration Plan

This repository contains Chef cookbooks that need individual migration planning:

### MODULE INVENTORY

- **nginx-multisite**:
    - Description: Configures Nginx web server with multiple SSL-enabled virtual hosts, security hardening, and self-signed certificates
    - Path: cookbooks/nginx-multisite
    - Technology: Chef
    - Key Features: Multi-site configuration, SSL certificate generation, security hardening (fail2ban, UFW firewall), sysctl security settings

- **cache**:
    - Description: Configures Redis and Memcached caching services with authentication
    - Path: cookbooks/cache
    - Technology: Chef
    - Key Features: Redis with password authentication, Memcached configuration, log directory management

- **fastapi-tutorial**:
    - Description: Deploys a FastAPI Python application with PostgreSQL database backend
    - Path: cookbooks/fastapi-tutorial
    - Technology: Chef
    - Key Features: Python virtual environment setup, Git repository deployment, PostgreSQL database creation, systemd service configuration

### Infrastructure Files

- `Berksfile`: Dependency management file listing cookbook dependencies (nginx, ssl_certificate, memcached, redisio)
- `Policyfile.rb`: Chef Policyfile defining the run list and cookbook dependencies
- `solo.json`: Configuration data for Chef Solo with site configurations and security settings
- `solo.rb`: Chef Solo configuration file
- `Vagrantfile`: Vagrant configuration for local development using Fedora 42
- `vagrant-provision.sh`: Bash script for provisioning the Vagrant VM with Chef

### Target Details

- **Operating System**: Fedora 42 (based on Vagrantfile), with support for Ubuntu 18.04+ and CentOS 7+ (based on cookbook metadata)
- **Virtual Machine Technology**: Vagrant with libvirt provider
- **Cloud Platform**: Not specified, appears to be designed for on-premises or generic cloud deployment

## Migration Approach

### Key Dependencies to Address

- **nginx (~> 12.0)**: Replace with Ansible nginx role or nginx_core module
- **memcached (~> 6.0)**: Replace with Ansible memcached role or package module
- **redisio (~> 7.2.4)**: Replace with Ansible redis role or package module
- **ssl_certificate (~> 2.1)**: Replace with Ansible openssl_* modules for certificate management

### Security Considerations

- **Firewall configuration**: Migrate UFW rules to Ansible ufw module
- **fail2ban setup**: Migrate fail2ban configuration to Ansible template
- **SSH hardening**: Preserve SSH security settings (disable root login, password authentication)
- **SSL certificate management**: Ensure proper handling of SSL certificates and private keys
- **Redis authentication**: Preserve Redis password configuration
- **PostgreSQL security**: Maintain database user permissions and password security
- **sysctl security settings**: Migrate sysctl security configurations

### Technical Challenges

- **SSL certificate generation**: The Chef cookbook generates self-signed certificates for each site. This needs to be replicated in Ansible using the openssl_* modules.
- **Multi-site configuration**: The dynamic generation of Nginx site configurations based on node attributes needs to be carefully migrated to Ansible templates and variables.
- **Service dependencies**: Ensuring proper ordering of service deployments (e.g., PostgreSQL before FastAPI application)
- **Idempotent database creation**: Ensuring PostgreSQL database creation remains idempotent in Ansible

### Migration Order

1. **nginx-multisite cookbook** (medium complexity)
   - Begin with core Nginx configuration
   - Implement security hardening (fail2ban, UFW)
   - Set up SSL certificate generation
   - Configure virtual hosts

2. **cache cookbook** (low complexity)
   - Set up Memcached
   - Configure Redis with authentication

3. **fastapi-tutorial cookbook** (medium complexity)
   - Set up PostgreSQL database
   - Deploy FastAPI application
   - Configure systemd service

### Assumptions

1. The target environment will continue to be Fedora 42 or a compatible Linux distribution.
2. Self-signed certificates are acceptable for development/testing purposes.
3. The same security hardening measures will be maintained in the Ansible configuration.
4. The FastAPI application repository at https://github.com/dibanez/fastapi_tutorial.git will remain available.
5. The Redis password "redis_secure_password_123" will need to be stored securely in Ansible Vault.
6. The PostgreSQL credentials (user: fastapi, password: fastapi_password) will need to be stored securely in Ansible Vault.
7. The current Chef implementation does not use encrypted data bags or other secret management, so no existing secrets need to be migrated.
8. The Vagrant development environment will be preserved or replaced with an equivalent Ansible-based setup.