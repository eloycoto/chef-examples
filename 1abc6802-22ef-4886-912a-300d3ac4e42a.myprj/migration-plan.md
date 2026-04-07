# MIGRATION FROM CHEF TO ANSIBLE

## Executive Summary

This repository contains a Chef-based infrastructure for deploying a multi-site Nginx configuration with caching services (Memcached and Redis) and a FastAPI application with PostgreSQL. The migration to Ansible is estimated to be of medium complexity, requiring approximately 3-4 weeks of effort for a single engineer or 1-2 weeks for a small team.

The repository consists of three main Chef cookbooks with clear responsibilities and minimal interdependencies, making it suitable for an incremental migration approach. The configuration includes security hardening, SSL certificate management, and service deployment, all of which have direct equivalents in Ansible.

## Module Migration Plan

This repository contains Chef cookbooks that need individual migration planning:

### MODULE INVENTORY

- **nginx-multisite**:
    - Description: Configures Nginx with multiple SSL-enabled virtual hosts, security hardening, and firewall configuration
    - Path: cookbooks/nginx-multisite
    - Technology: Chef
    - Key Features: Multi-site configuration, SSL certificate generation, UFW firewall, Fail2ban integration, SSH hardening

- **cache**:
    - Description: Configures caching services including Memcached and Redis with authentication
    - Path: cookbooks/cache
    - Technology: Chef
    - Key Features: Redis with password authentication, Memcached configuration, service management

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
- `vagrant-provision.sh`: Provisioning script for Vagrant VM setup

### Target Details

- **Operating System**: Fedora (based on Vagrantfile using "generic/fedora42"), with support for Ubuntu 18.04+ and CentOS 7+ (based on cookbook metadata)
- **Virtual Machine Technology**: Vagrant with libvirt provider
- **Cloud Platform**: Not specified, appears to be designed for on-premises or generic cloud deployment

## Migration Approach

### Key Dependencies to Address

- **nginx (~> 12.0)**: Replace with Ansible nginx role or nginx_config module
- **memcached (~> 6.0)**: Replace with Ansible memcached role or package/service modules
- **redisio (~> 7.2.4)**: Replace with Ansible redis role or package/service modules
- **ssl_certificate (~> 2.1)**: Replace with Ansible openssl_* modules for certificate management

### Security Considerations

- **Firewall (UFW)**: Migrate to Ansible ufw module or firewalld module depending on target OS
- **Fail2ban**: Migrate to Ansible fail2ban module or template configuration
- **SSH hardening**: Use Ansible to manage sshd_config with lineinfile or template module
- **SSL certificates**: Use Ansible openssl_certificate module for self-signed certificates
- **Redis password**: Store Redis password in Ansible Vault for secure management
- **PostgreSQL credentials**: Store database credentials in Ansible Vault

### Technical Challenges

- **Multi-site configuration**: Ensure proper templating of Nginx site configurations with Ansible
- **Service dependencies**: Maintain proper ordering of service installations and configurations
- **SSL certificate management**: Ensure proper permissions and ownership of SSL certificates and keys
- **Database initialization**: Ensure idempotent database creation and user setup

### Migration Order

1. **nginx-multisite cookbook** (moderate complexity, foundation for web services)
   - Start with basic Nginx installation and configuration
   - Add security components (fail2ban, ufw, SSH hardening)
   - Implement SSL certificate generation
   - Configure virtual hosts

2. **cache cookbook** (low complexity, independent service)
   - Implement Memcached configuration
   - Implement Redis with authentication

3. **fastapi-tutorial cookbook** (high complexity, application deployment)
   - Set up PostgreSQL database
   - Deploy FastAPI application
   - Configure systemd service

### Assumptions

1. The target environment will continue to be Fedora-based, but with compatibility for Ubuntu and CentOS as specified in the cookbook metadata.
2. Self-signed certificates are acceptable for development/testing, but production would likely require proper certificates.
3. The current security configurations (fail2ban, ufw, SSH hardening) are appropriate for the target environment.
4. The FastAPI application repository at https://github.com/dibanez/fastapi_tutorial.git will remain available.
5. The Redis password and PostgreSQL credentials in the current configuration are for development only and will be replaced with secure values stored in Ansible Vault.
6. The current Vagrant setup is for development only and may not be needed in the Ansible migration.
7. No custom Ohai plugins or Chef handlers are in use that would require special handling.
8. No external Chef server is being used; the configuration is designed for Chef Solo.