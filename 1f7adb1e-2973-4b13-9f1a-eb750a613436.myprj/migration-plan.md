# MIGRATION FROM CHEF TO ANSIBLE

## Executive Summary

This repository contains a Chef-based infrastructure setup for a multi-site Nginx web server with caching services (Redis and Memcached) and a FastAPI application backed by PostgreSQL. The migration to Ansible is estimated to be of medium complexity, requiring approximately 3-4 weeks for a complete transition with testing. The repository has a well-structured organization with clear separation of concerns between different cookbooks, making it amenable to incremental migration.

## Module Migration Plan

This repository contains Chef cookbooks that need individual migration planning:

### MODULE INVENTORY

- **nginx-multisite**:
    - Description: Nginx web server with multiple SSL-enabled virtual hosts, security hardening, and site configuration
    - Path: cookbooks/nginx-multisite
    - Technology: Chef
    - Key Features: Multi-site configuration, SSL certificate generation, security hardening (fail2ban, ufw)

- **cache**:
    - Description: Caching services configuration including Memcached and Redis with authentication
    - Path: cookbooks/cache
    - Technology: Chef
    - Key Features: Redis with password authentication, Memcached configuration

- **fastapi-tutorial**:
    - Description: Python FastAPI application deployment with PostgreSQL database backend
    - Path: cookbooks/fastapi-tutorial
    - Technology: Chef
    - Key Features: Python virtual environment setup, PostgreSQL database creation, systemd service configuration

### Infrastructure Files

- `Berksfile`: Dependency management file listing cookbook dependencies (nginx, memcached, redisio, ssl_certificate)
- `Policyfile.rb`: Chef policy file defining the run list and cookbook dependencies
- `solo.json`: Configuration data for Chef Solo with site configurations and security settings
- `solo.rb`: Chef Solo configuration file
- `Vagrantfile`: Vagrant configuration for local development using Fedora 42
- `vagrant-provision.sh`: Provisioning script for Vagrant to install and run Chef

### Target Details

- **Operating System**: Fedora (based on Vagrantfile specifying "generic/fedora42")
- **Virtual Machine Technology**: Vagrant with libvirt provider
- **Cloud Platform**: Not specified, appears to be designed for on-premises or local development

## Migration Approach

### Key Dependencies to Address

- **nginx (~> 12.0)**: Replace with Ansible nginx role or community.general.nginx_* modules
- **memcached (~> 6.0)**: Replace with Ansible memcached role or package installation tasks
- **redisio (~> 7.2.4)**: Replace with Ansible redis role or package installation tasks
- **ssl_certificate (~> 2.1)**: Replace with Ansible openssl_* modules for certificate generation

### Security Considerations

- **fail2ban configuration**: Migrate using Ansible's package and template modules
- **ufw firewall rules**: Replace with Ansible's ufw module or firewalld for Fedora
- **SSH hardening**: Migrate using Ansible's lineinfile or template modules for sshd_config
- **SSL certificate generation**: Use Ansible's openssl_* modules for self-signed certificates
- **Redis password**: Store in Ansible Vault for secure credential management
- **PostgreSQL credentials**: Store database credentials in Ansible Vault

### Technical Challenges

- **Multi-site configuration**: Ensure proper templating of Nginx site configurations with Ansible's template module
- **Service dependencies**: Maintain proper ordering of service installations and configurations
- **SSL certificate management**: Ensure proper permissions and ownership for SSL certificates and keys
- **Python environment setup**: Properly configure Python virtual environments and dependencies
- **Database initialization**: Ensure idempotent database and user creation

### Migration Order

1. **nginx-multisite cookbook** (moderate complexity)
   - Start with basic Nginx installation and configuration
   - Add security hardening (fail2ban, ufw)
   - Configure SSL certificate generation
   - Set up virtual hosts

2. **cache cookbook** (low complexity)
   - Set up Memcached service
   - Configure Redis with authentication

3. **fastapi-tutorial cookbook** (high complexity)
   - Set up PostgreSQL database
   - Configure Python environment
   - Deploy FastAPI application
   - Configure systemd service

### Assumptions

1. The target environment will continue to be Fedora-based systems
2. Self-signed certificates are acceptable for development/testing
3. The same directory structure for web content will be maintained
4. The same security hardening measures will be required
5. Redis and Memcached configurations will remain similar
6. PostgreSQL database structure and credentials can remain the same
7. The FastAPI application source will be pulled from the same Git repository
8. The migration will be tested in a Vagrant environment before production deployment

## Detailed Migration Tasks

### 1. Infrastructure Setup

- Create Ansible project structure with roles, playbooks, and inventory
- Set up Ansible Vault for secrets management
- Create equivalent Vagrant configuration for testing

### 2. Nginx Multi-site Migration

- Create nginx role with tasks for installation and base configuration
- Develop templates for nginx.conf and security.conf
- Create tasks for SSL certificate generation
- Develop templates for virtual host configurations
- Implement security hardening with fail2ban and ufw

### 3. Caching Services Migration

- Create memcached role for installation and configuration
- Create redis role with secure password configuration
- Ensure proper service management and startup

### 4. FastAPI Application Migration

- Create postgresql role for database installation and configuration
- Develop python role for environment setup
- Create fastapi role for application deployment
- Implement systemd service configuration

### 5. Testing and Validation

- Create test playbooks for each component
- Validate configurations against original Chef implementations
- Test full stack deployment in Vagrant environment
- Document any differences or improvements

## Conclusion

The migration from Chef to Ansible for this infrastructure is straightforward but requires careful attention to security configurations and service dependencies. The well-structured nature of the existing Chef cookbooks provides a clear path for migration. The recommended approach is to migrate one cookbook at a time, starting with the nginx-multisite cookbook, followed by the cache cookbook, and finally the fastapi-tutorial cookbook.