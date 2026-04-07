# MIGRATION FROM CHEF TO ANSIBLE

## Executive Summary

This repository contains a Chef-based infrastructure setup for a multi-site Nginx web server with caching services (Redis and Memcached) and a FastAPI application backed by PostgreSQL. The migration to Ansible is estimated to be of medium complexity, requiring approximately 3-4 weeks for a complete transition. The repository has a well-structured organization with clear separation of concerns between cookbooks, making it amenable to an incremental migration approach.

## Module Migration Plan

This repository contains Chef cookbooks that need individual migration planning:

### MODULE INVENTORY

- **nginx-multisite**:
    - Description: Configures Nginx web server with multiple SSL-enabled virtual hosts, security hardening, and site-specific configurations
    - Path: cookbooks/nginx-multisite
    - Technology: Chef
    - Key Features: Multi-site configuration, SSL certificate generation, security hardening with fail2ban and UFW

- **cache**:
    - Description: Configures caching services including Memcached and Redis with authentication
    - Path: cookbooks/cache
    - Technology: Chef
    - Key Features: Redis with password authentication, Memcached configuration

- **fastapi-tutorial**:
    - Description: Deploys a FastAPI Python application with PostgreSQL database backend
    - Path: cookbooks/fastapi-tutorial
    - Technology: Chef
    - Key Features: Python virtual environment setup, PostgreSQL database creation, systemd service configuration

### Infrastructure Files

- `Berksfile`: Defines cookbook dependencies including external cookbooks from Chef Supermarket (nginx, memcached, redisio)
- `Policyfile.rb`: Defines the run list and cookbook dependencies for Chef Policyfile workflow
- `solo.json`: Contains node attributes for Chef Solo, including Nginx site configurations and security settings
- `solo.rb`: Chef Solo configuration file defining cookbook paths and log settings
- `Vagrantfile`: Defines a Vagrant VM for development and testing with Fedora 42
- `vagrant-provision.sh`: Shell script for provisioning the Vagrant VM with Chef

### Target Details

- **Operating System**: Fedora 42 (based on Vagrantfile), with support for Ubuntu 18.04+ and CentOS 7+ (based on cookbook metadata)
- **Virtual Machine Technology**: Libvirt (based on Vagrantfile configuration)
- **Cloud Platform**: Not specified, appears to be targeting on-premises or generic VM deployment

## Migration Approach

### Key Dependencies to Address

- **nginx (~> 12.0)**: Replace with Ansible nginx role from Ansible Galaxy or create custom role
- **memcached (~> 6.0)**: Replace with Ansible memcached role or tasks
- **redisio (~> 7.2.4)**: Replace with Ansible Redis role or tasks
- **ssl_certificate (~> 2.1)**: Replace with Ansible certificate management tasks using openssl module

### Security Considerations

- **SSL Certificate Management**: Migration must preserve self-signed certificate generation for development environments
- **Fail2ban Configuration**: Ensure fail2ban rules are properly migrated to Ansible
- **UFW Firewall Rules**: Migrate UFW firewall configuration to Ansible ufw module
- **SSH Hardening**: Preserve SSH security settings (disable root login, password authentication)
- **Redis Authentication**: Ensure Redis password is securely managed in Ansible Vault
- **PostgreSQL Credentials**: Store database credentials in Ansible Vault

### Technical Challenges

- **Multi-site Nginx Configuration**: The dynamic generation of multiple virtual hosts with SSL needs careful translation to Ansible templates
- **Service Orchestration**: Ensuring proper service restart handlers and notifications are maintained in Ansible
- **PostgreSQL User/Database Creation**: Ensuring idempotent database operations in Ansible
- **Python Environment Management**: Properly setting up virtualenv and dependencies in Ansible

### Migration Order

1. **nginx-multisite cookbook** (moderate complexity, foundation for web services)
   - Start with basic Nginx installation and configuration
   - Add SSL certificate generation
   - Implement security hardening (fail2ban, UFW)
   - Configure virtual hosts

2. **cache cookbook** (low complexity, independent service)
   - Implement Memcached configuration
   - Implement Redis with authentication

3. **fastapi-tutorial cookbook** (high complexity, application deployment)
   - Set up PostgreSQL database
   - Configure Python environment and dependencies
   - Deploy FastAPI application
   - Configure systemd service

### Assumptions

1. The target environment will continue to be Fedora 42 or compatible Linux distributions
2. Self-signed certificates are acceptable for development environments
3. The same security hardening measures will be maintained
4. The FastAPI application source will continue to be pulled from the same Git repository
5. Redis and PostgreSQL passwords in the Chef recipes are development passwords and will be replaced with Ansible Vault secured passwords
6. The multi-site configuration pattern will be maintained with the same domain structure
7. The current Chef Solo workflow suggests a push-based deployment model which will be maintained in Ansible