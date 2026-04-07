# Migration Plan: nginx-multisite

**TLDR**: This cookbook configures a secure Nginx web server with multiple virtual hosts (3 sites), each with SSL enabled. It includes security hardening via fail2ban, UFW firewall, SSH hardening, and system-level security configurations.

## Service Type and Instances

**Service Type**: Web Server

**Configured Instances**:

- **test.cluster.local**: Main test website
  - Location/Path: /opt/server/test
  - Port/Socket: 80 (redirect to 443), 443 (SSL)
  - Key Config: SSL enabled, HTTP to HTTPS redirect

- **ci.cluster.local**: Continuous integration website
  - Location/Path: /opt/server/ci
  - Port/Socket: 80 (redirect to 443), 443 (SSL)
  - Key Config: SSL enabled, HTTP to HTTPS redirect

- **status.cluster.local**: Status monitoring website
  - Location/Path: /opt/server/status
  - Port/Socket: 80 (redirect to 443), 443 (SSL)
  - Key Config: SSL enabled, HTTP to HTTPS redirect

## File Structure

```
cookbooks/nginx-multisite/recipes/default.rb
cookbooks/nginx-multisite/recipes/nginx.rb
cookbooks/nginx-multisite/recipes/security.rb
cookbooks/nginx-multisite/recipes/sites.rb
cookbooks/nginx-multisite/recipes/ssl.rb
cookbooks/nginx-multisite/templates/default/fail2ban.jail.local.erb
cookbooks/nginx-multisite/templates/default/nginx.conf.erb
cookbooks/nginx-multisite/templates/default/security.conf.erb
cookbooks/nginx-multisite/templates/default/site.conf.erb
cookbooks/nginx-multisite/templates/default/sysctl-security.conf.erb
cookbooks/nginx-multisite/attributes/default.rb
```

## Module Explanation

The cookbook performs operations in this order:

1. **default** (`cookbooks/nginx-multisite/recipes/default.rb`):
   - Includes other recipes in sequence: security, nginx, ssl, sites
   - Resources: include_recipe (4)

2. **security** (`cookbooks/nginx-multisite/recipes/security.rb`):
   - Installs security packages: fail2ban, ufw
   - Configures fail2ban with custom jail settings
   - Sets up UFW firewall with default deny policy and specific allow rules
   - Configures system-level security via sysctl
   - Hardens SSH configuration if enabled in attributes
   - Resources: package (1), service (2), template (2), execute (8)
   - Conditional execution:
     - If node['security']['ssh']['disable_root'] is true:
       - Disables root SSH login
     - If node['security']['ssh']['password_auth'] is false:
       - Disables password authentication for SSH

3. **nginx** (`cookbooks/nginx-multisite/recipes/nginx.rb`):
   - Installs nginx package
   - Configures main nginx.conf with server settings
   - Adds security-specific configuration
   - Creates document root directories for each site
   - Places index.html files in each site's document root
   - Resources: package (1), template (2), service (1), directory (3), cookbook_file (3)
   - Creates document root directory for test.cluster.local
   - Places index.html file in test.cluster.local document root
   - Creates document root directory for ci.cluster.local
   - Places index.html file in ci.cluster.local document root
   - Creates document root directory for status.cluster.local
   - Places index.html file in status.cluster.local document root

4. **ssl** (`cookbooks/nginx-multisite/recipes/ssl.rb`):
   - Installs SSL-related packages: openssl, ca-certificates
   - Creates SSL certificate group
   - Creates directories for certificates and private keys
   - Generates self-signed SSL certificates for each site
   - Resources: package (1), group (1), directory (2), execute (3)
   - Generates self-signed SSL certificate for test.cluster.local
   - Sets proper permissions on test.cluster.local key files
   - Generates self-signed SSL certificate for ci.cluster.local
   - Sets proper permissions on ci.cluster.local key files
   - Generates self-signed SSL certificate for status.cluster.local
   - Sets proper permissions on status.cluster.local key files

5. **sites** (`cookbooks/nginx-multisite/recipes/sites.rb`):
   - Creates Nginx site configuration files for each site
   - Creates symlinks to enable sites
   - Removes default site configuration
   - Resources: template (3), link (3), file (1)
   - Creates site configuration in sites-available for test.cluster.local
   - Creates symlink in sites-enabled for test.cluster.local
   - Creates site configuration in sites-available for ci.cluster.local
   - Creates symlink in sites-enabled for ci.cluster.local
   - Creates site configuration in sites-available for status.cluster.local
   - Creates symlink in sites-enabled for status.cluster.local

## Dependencies

**External cookbook dependencies**: None specified
**System package dependencies**: nginx, fail2ban, ufw, openssl, ca-certificates
**Service dependencies**: nginx, fail2ban, ssh

## Checks for the Migration

**Files to verify**:
- /etc/nginx/nginx.conf
- /etc/nginx/conf.d/security.conf
- /etc/nginx/sites-available/test.cluster.local
- /etc/nginx/sites-available/ci.cluster.local
- /etc/nginx/sites-available/status.cluster.local
- /etc/nginx/sites-enabled/test.cluster.local
- /etc/nginx/sites-enabled/ci.cluster.local
- /etc/nginx/sites-enabled/status.cluster.local
- /etc/ssl/certs/test.cluster.local.crt
- /etc/ssl/certs/ci.cluster.local.crt
- /etc/ssl/certs/status.cluster.local.crt
- /etc/ssl/private/test.cluster.local.key
- /etc/ssl/private/ci.cluster.local.key
- /etc/ssl/private/status.cluster.local.key
- /etc/fail2ban/jail.local
- /etc/sysctl.d/99-security.conf
- /opt/server/test/index.html
- /opt/server/ci/index.html
- /opt/server/status/index.html

**Service endpoints to check**:
- Ports listening: 80, 443
- Network interfaces: All interfaces (default)

**Templates rendered**:
- nginx.conf.erb → /etc/nginx/nginx.conf (1 time)
- security.conf.erb → /etc/nginx/conf.d/security.conf (1 time)
- site.conf.erb → /etc/nginx/sites-available/test.cluster.local (1 time)
- site.conf.erb → /etc/nginx/sites-available/ci.cluster.local (1 time)
- site.conf.erb → /etc/nginx/sites-available/status.cluster.local (1 time)
- fail2ban.jail.local.erb → /etc/fail2ban/jail.local (1 time)
- sysctl-security.conf.erb → /etc/sysctl.d/99-security.conf (1 time)

## Pre-flight checks:

```bash
# Service status
systemctl status nginx
systemctl status fail2ban

# Process verification
ps aux | grep nginx
ps aux | grep fail2ban

# Configuration validation
nginx -t
fail2ban-client ping

# Site availability - test.cluster.local
curl -I -k https://test.cluster.local  # Should return 200 OK
curl -I http://test.cluster.local  # Should return 301 redirect to HTTPS
openssl s_client -connect test.cluster.local:443 -servername test.cluster.local </dev/null 2>/dev/null | grep "Verify return code"

# Site availability - ci.cluster.local
curl -I -k https://ci.cluster.local  # Should return 200 OK
curl -I http://ci.cluster.local  # Should return 301 redirect to HTTPS
openssl s_client -connect ci.cluster.local:443 -servername ci.cluster.local </dev/null 2>/dev/null | grep "Verify return code"

# Site availability - status.cluster.local
curl -I -k https://status.cluster.local  # Should return 200 OK
curl -I http://status.cluster.local  # Should return 301 redirect to HTTPS
openssl s_client -connect status.cluster.local:443 -servername status.cluster.local </dev/null 2>/dev/null | grep "Verify return code"

# SSL certificate verification
openssl x509 -in /etc/ssl/certs/test.cluster.local.crt -text -noout | grep Subject
openssl x509 -in /etc/ssl/certs/ci.cluster.local.crt -text -noout | grep Subject
openssl x509 -in /etc/ssl/certs/status.cluster.local.crt -text -noout | grep Subject

# File permissions
ls -la /etc/ssl/private/test.cluster.local.key  # Should be 640 root:ssl-cert
ls -la /etc/ssl/private/ci.cluster.local.key  # Should be 640 root:ssl-cert
ls -la /etc/ssl/private/status.cluster.local.key  # Should be 640 root:ssl-cert

# Firewall status
ufw status
ufw status verbose  # Should show default deny with allow for 22, 80, 443

# Fail2ban status
fail2ban-client status
fail2ban-client status sshd
fail2ban-client status nginx-http-auth
fail2ban-client status nginx-limit-req
fail2ban-client status nginx-botsearch

# SSH hardening verification
grep "PermitRootLogin" /etc/ssh/sshd_config  # Should show "PermitRootLogin no"
grep "PasswordAuthentication" /etc/ssh/sshd_config  # Should show "PasswordAuthentication no"

# Sysctl security settings
sysctl -a | grep "net.ipv4.conf.all.rp_filter"  # Should be 1
sysctl -a | grep "net.ipv4.conf.all.accept_redirects"  # Should be 0
sysctl -a | grep "net.ipv4.tcp_syncookies"  # Should be 1

# Document roots
ls -la /opt/server/test/
ls -la /opt/server/ci/
ls -la /opt/server/status/

# Nginx logs
tail -n 20 /var/log/nginx/test.cluster.local_access.log
tail -n 20 /var/log/nginx/test.cluster.local_error.log
tail -n 20 /var/log/nginx/ci.cluster.local_access.log
tail -n 20 /var/log/nginx/ci.cluster.local_error.log
tail -n 20 /var/log/nginx/status.cluster.local_access.log
tail -n 20 /var/log/nginx/status.cluster.local_error.log

# Network listening
netstat -tulpn | grep nginx  # Should show ports 80 and 443
ss -tlnp | grep nginx  # Alternative check for ports 80 and 443
lsof -i :80
lsof -i :443
```