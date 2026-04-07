# MIGRATION FROM CHEF TO ANSIBLE

## Executive Summary

This repository contains a Chef-based infrastructure setup for a multi-site Nginx web server with caching services (Redis and Memcached) and a FastAPI application with PostgreSQL. The migration to Ansible is estimated to be of medium complexity, requiring approximately 3-4 weeks for a complete transition. The repository is well-structured with clear separation of concerns, which will facilitate a smooth migration process.

## Module Migration Plan

This repository contains Chef cookbooks that need individual migration planning:

### MODULE INVENTORY

- **nginx-multisite**:
    - Description: Nginx web server with multiple SSL-enabled virtual hosts, security hardening, and site configuration
    - Path: cookbooks/nginx-multisite
    - Technology: Chef
    - Key Features: Multi-site configuration, SSL certificate generation, security hardening (fail2ban, UFW firewall)

- **cache**:
    - Description: Caching services configuration including Memcached and Redis with authentication
    - Path: cookbooks/cache
    - Technology: Chef
    - Key Features: Redis with password authentication, Memcached configuration

- **fastapi-tutorial**:
    - Description: Python FastAPI application deployment with PostgreSQL database
    - Path: cookbooks/fastapi-tutorial
    - Technology: Chef
    - Key Features: Python virtual environment setup, PostgreSQL database creation, systemd service configuration

### Infrastructure Files

- `Berksfile`: Dependency management file for Chef cookbooks, lists both local and external cookbook dependencies
- `Policyfile.rb`: Chef policy file defining the run list and cookbook dependencies
- `Policyfile.lock.json`: Locked versions of cookbook dependencies
- `solo.json`: Configuration data for Chef Solo, contains site configurations and security settings
- `solo.rb`: Chef Solo configuration file
- `Vagrantfile`: Vagrant configuration for local development/testing using Fedora 42
- `vagrant-provision.sh`: Shell script for provisioning the Vagrant VM with Chef

### Target Details

- **Operating System**: Fedora 42 (based on Vagrantfile configuration)
- **Virtual Machine Technology**: libvirt (based on Vagrantfile provider configuration)
- **Cloud Platform**: Not specified, appears to be designed for on-premises or generic cloud deployment

## Migration Approach

### Key Dependencies to Address

- **nginx (~> 12.0)**: Replace with Ansible nginx role or direct package installation
- **memcached (~> 6.0)**: Replace with Ansible memcached role or direct package configuration
- **redisio (~> 7.2.4)**: Replace with Ansible redis role or direct package configuration
- **ssl_certificate (~> 2.1)**: Replace with Ansible's openssl_* modules for certificate management

### Security Considerations

- **Firewall configuration**: Migrate UFW rules to Ansible's ufw module
- **fail2ban setup**: Use Ansible's template module to configure fail2ban
- **SSH hardening**: Migrate SSH security configurations using Ansible's lineinfile or template modules
- **SSL certificate management**: Use Ansible's openssl_* modules for certificate generation and management
- **Redis authentication**: Ensure Redis password is stored securely in Ansible Vault

### Technical Challenges

- **Multi-site Nginx configuration**: Ensure proper templating of multiple virtual hosts with SSL
- **PostgreSQL user and database creation**: Migrate to Ansible's postgresql_* modules
- **Redis configuration workarounds**: The Chef cookbook contains a hack to fix Redis configuration, which needs careful migration
- **Service dependencies**: Maintain proper ordering of service installations and configurations

### Migration Order

1. **nginx-multisite cookbook** (moderate complexity)
   - Start with basic Nginx installation and configuration
   - Add SSL certificate generation
   - Configure virtual hosts
   - Implement security hardening

2. **cache cookbook** (low complexity)
   - Set up Memcached
   - Configure Redis with authentication

3. **fastapi-tutorial cookbook** (high complexity)
   - Set up PostgreSQL
   - Deploy Python application
   - Configure systemd service

### Assumptions

1. The target environment will continue to be Fedora-based systems
2. SSL certificates will remain self-signed for development purposes
3. The same security hardening measures will be maintained
4. The FastAPI application source will continue to be pulled from the same Git repository
5. Redis and Memcached configurations will remain similar
6. The multi-site configuration pattern will be preserved
7. PostgreSQL database credentials will need to be secured in Ansible Vault
8. Redis authentication password will need to be secured in Ansible Vault