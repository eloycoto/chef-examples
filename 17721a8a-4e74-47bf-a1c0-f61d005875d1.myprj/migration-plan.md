# MIGRATION FROM CHEF TO ANSIBLE

## Executive Summary

This repository contains a Chef-based infrastructure for deploying a multi-site Nginx server with caching services (Memcached and Redis) and a FastAPI application with PostgreSQL. The migration to Ansible is estimated to be of medium complexity, requiring approximately 2-3 weeks of effort for a skilled Ansible developer.

The repository consists of three main Chef cookbooks:
1. **nginx-multisite**: Configures Nginx with multiple SSL-enabled virtual hosts and security hardening
2. **cache**: Sets up Memcached and Redis caching services
3. **fastapi-tutorial**: Deploys a Python FastAPI application with PostgreSQL database

The migration will require careful handling of SSL certificates, security configurations, and service dependencies. The existing Chef cookbooks follow good practices with clear separation of concerns, which will facilitate a straightforward migration to Ansible roles.

## Module Migration Plan

This repository contains Chef cookbooks that need individual migration planning:

### MODULE INVENTORY

- **nginx-multisite**:
    - Description: Nginx web server with multiple SSL-enabled virtual hosts, security hardening (fail2ban, ufw firewall), and self-signed SSL certificate generation
    - Path: cookbooks/nginx-multisite
    - Technology: Chef
    - Key Features: Multi-site configuration, SSL certificate management, security hardening (fail2ban, ufw, sysctl)

- **cache**:
    - Description: Caching services configuration including Memcached and Redis with authentication
    - Path: cookbooks/cache
    - Technology: Chef
    - Key Features: Redis with password authentication, Memcached configuration

- **fastapi-tutorial**:
    - Description: Python FastAPI application deployment with PostgreSQL database
    - Path: cookbooks/fastapi-tutorial
    - Technology: Chef
    - Key Features: Git-based deployment, Python virtual environment, PostgreSQL database setup, systemd service configuration

### Infrastructure Files

- `Berksfile`: Defines cookbook dependencies including nginx (~> 12.0), memcached (~> 6.0), and redisio (~> 7.2.4)
- `Policyfile.rb`: Defines the Chef policy with run list and cookbook dependencies
- `solo.json`: Contains node configuration including Nginx site definitions and security settings
- `solo.rb`: Chef Solo configuration file
- `Vagrantfile`: Defines a Fedora 42 VM for development/testing with port forwarding and networking
- `vagrant-provision.sh`: Shell script to provision the Vagrant VM with Chef

### Target Details

Based on the source configuration files:

- **Operating System**: Supports both Ubuntu (>= 18.04) and CentOS (>= 7.0), with the development environment using Fedora 42
- **Virtual Machine Technology**: Vagrant with libvirt provider
- **Cloud Platform**: Not specified, appears to be designed for on-premises or generic cloud deployment

## Migration Approach

### Key Dependencies to Address

- **nginx (~> 12.0)**: Replace with Ansible nginx role or nginx_core module
- **memcached (~> 6.0)**: Replace with Ansible memcached role or package installation
- **redisio (~> 7.2.4)**: Replace with Ansible redis role or package installation
- **ssl_certificate (~> 2.1)**: Replace with Ansible openssl_* modules for certificate generation

### Security Considerations

- **fail2ban configuration**: Migrate using Ansible's package and template modules
- **ufw firewall rules**: Replace with Ansible's ufw module
- **SSH hardening**: Migrate using Ansible's lineinfile or template module for sshd_config
- **sysctl security settings**: Migrate using Ansible's sysctl module
- **Redis password**: Store in Ansible Vault and reference in templates
- **PostgreSQL credentials**: Store in Ansible Vault and reference in templates

### Technical Challenges

- **Self-signed SSL certificates**: Ensure proper handling of certificate generation and permissions using Ansible's openssl_* modules
- **Redis configuration**: The Chef cookbook uses a ruby_block to modify Redis configuration files; this will need to be replaced with proper Ansible templates
- **Service dependencies**: Ensure proper ordering of service installation and configuration, particularly for the FastAPI application which depends on PostgreSQL

### Migration Order

1. **nginx-multisite cookbook** (medium complexity)
   - Base Nginx installation and configuration
   - SSL certificate generation
   - Virtual host configuration
   - Security hardening (fail2ban, ufw, sysctl)

2. **cache cookbook** (low complexity)
   - Memcached installation and configuration
   - Redis installation and configuration with authentication

3. **fastapi-tutorial cookbook** (medium complexity)
   - PostgreSQL installation and database setup
   - Python environment setup
   - Application deployment
   - Systemd service configuration

### Assumptions

1. The target environment will continue to be either Ubuntu (>= 18.04) or CentOS (>= 7.0)
2. The same security requirements will apply in the Ansible implementation
3. Self-signed certificates are acceptable for development; production would likely use different certificate sources
4. The FastAPI application source repository will remain available at the specified URL
5. The Redis password and PostgreSQL credentials will need to be managed securely in the Ansible implementation
6. The Vagrant development environment will be maintained for testing the Ansible playbooks