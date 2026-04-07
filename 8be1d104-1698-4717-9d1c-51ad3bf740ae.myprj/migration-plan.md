# MIGRATION FROM CHEF TO ANSIBLE

## Executive Summary

This repository contains a Chef-based infrastructure setup for a multi-site Nginx configuration with caching services (Redis and Memcached) and a FastAPI application with PostgreSQL. The migration to Ansible is estimated to be of medium complexity, requiring approximately 3-4 weeks for a complete transition. The repository is well-structured with clear separation of concerns, which will facilitate a smooth migration process.

## Module Migration Plan

This repository contains Chef cookbooks that need individual migration planning:

### MODULE INVENTORY

- **nginx-multisite**:
    - Description: Configures Nginx with multiple SSL-enabled virtual hosts, security hardening, and site-specific configurations
    - Path: cookbooks/nginx-multisite
    - Technology: Chef
    - Key Features: Multi-site SSL configuration, security hardening (fail2ban, ufw), self-signed certificate generation

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
- `solo.json`: Contains node attributes for Nginx sites configuration and security settings
- `solo.rb`: Chef Solo configuration file defining cookbook paths and log settings
- `Vagrantfile`: Defines a Fedora 42 VM for local development and testing
- `vagrant-provision.sh`: Bash script to provision the Vagrant VM with Chef

### Target Details

- **Operating System**: Fedora 42 (based on Vagrantfile), with support for Ubuntu 18.04+ and CentOS 7+ (based on cookbook metadata)
- **Virtual Machine Technology**: Libvirt (based on Vagrantfile configuration)
- **Cloud Platform**: Not specified, appears to be designed for on-premises or generic cloud deployment

## Migration Approach

### Key Dependencies to Address

- **nginx (~> 12.0)**: Replace with Ansible nginx role from Ansible Galaxy or create a custom role
- **memcached (~> 6.0)**: Use Ansible memcached role or create tasks for memcached installation and configuration
- **redisio (~> 7.2.4)**: Use Ansible Redis role or create tasks for Redis installation and configuration
- **ssl_certificate (~> 2.1)**: Replace with Ansible OpenSSL module for certificate generation

### Security Considerations

- **Firewall (ufw)**: Migrate to Ansible's `ufw` module for firewall configuration
- **Fail2ban**: Create Ansible tasks for fail2ban installation and configuration
- **SSH hardening**: Use Ansible to configure SSH security settings (disable root login, password authentication)
- **SSL/TLS**: Ensure secure certificate generation and configuration in Ansible
- **Redis authentication**: Securely manage Redis password in Ansible Vault
- **PostgreSQL credentials**: Store database credentials in Ansible Vault

### Technical Challenges

- **Multi-site Nginx configuration**: Create a flexible Ansible role that can handle multiple virtual hosts with SSL
- **Self-signed certificates**: Implement certificate generation logic in Ansible
- **Service dependencies**: Ensure proper ordering of service installation and configuration
- **Python environment setup**: Create idempotent tasks for Python virtual environment creation and package installation

### Migration Order

1. **nginx-multisite cookbook** (moderate complexity, foundation for web services)
   - Start with basic Nginx installation and configuration
   - Add security hardening (fail2ban, ufw)
   - Implement SSL certificate generation
   - Configure virtual hosts

2. **cache cookbook** (low complexity, independent service)
   - Implement Memcached configuration
   - Implement Redis installation and configuration with authentication

3. **fastapi-tutorial cookbook** (high complexity, depends on PostgreSQL)
   - Set up PostgreSQL database
   - Configure Python environment
   - Deploy FastAPI application
   - Create systemd service

### Assumptions

1. The migration will maintain the same functionality and security posture as the original Chef implementation
2. The target environment will continue to be Fedora/CentOS/Ubuntu based on the current support
3. Self-signed certificates are acceptable for the migrated solution (production would likely use Let's Encrypt or other CA)
4. The Redis password "redis_secure_password_123" in the Chef cookbook is a placeholder and will be replaced with a secure password in Ansible Vault
5. The PostgreSQL credentials in the FastAPI cookbook are development credentials and will be replaced with secure credentials in Ansible Vault
6. The Git repository URL for the FastAPI application will remain accessible
7. The current directory structure with multiple sites will be maintained in the Ansible implementation
8. The Vagrant development environment will be migrated to use Ansible provisioning instead of Chef