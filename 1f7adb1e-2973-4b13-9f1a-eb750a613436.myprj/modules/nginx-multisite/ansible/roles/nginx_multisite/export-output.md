Migration Summary for nginx_multisite:
  Total items: 22
  Completed: 22
  Pending: 0
  Missing: 0
  Errors: 0
  Write attempts: 1
  Validation attempts: 0

Final Validation Report:
All migration tasks have been completed successfully

Validation passed with warnings:
ansible-lint: Passed with 1 warning(s):
[HIGH] handlers/main.yml:17 [no-changed-when] Commands should not change things if nothing needs doing. (Task/Handler: Reload sysctl)

==============================
Rule Hints (How to Fix):
==============================
# no-changed-when

Commands should use `changed_when` to indicate when they actually change something.

## Problematic code

```yaml
- name: Does not handle any output or return codes
  ansible.builtin.command: cat {{ my_file | quote }}
```

## Correct code

```yaml
- name: Handle command output
  ansible.builtin.command: cat {{ my_file | quote }}
  register: my_output
  changed_when: my_output.rc != 0
```

Common patterns:
- `changed_when: false` - Task never changes anything
- `changed_when: true` - Task always changes something
- `changed_when: result.rc != 0` - Use command result to determine change

Final checklist:
## Checklist: nginx_multisite

### Templates
- [x] cookbooks/nginx-multisite/templates/default/fail2ban.jail.local.erb → ./ansible/roles/nginx_multisite/templates/fail2ban.jail.local.j2 (complete) - Converted fail2ban.jail.local.erb to fail2ban.jail.local.j2 template.
- [x] cookbooks/nginx-multisite/templates/default/nginx.conf.erb → ./ansible/roles/nginx_multisite/templates/nginx.conf.j2 (complete) - Converted nginx.conf from ERB to Jinja2 template. No variables were present in the original template.
- [x] cookbooks/nginx-multisite/templates/default/security.conf.erb → ./ansible/roles/nginx_multisite/templates/security.conf.j2 (complete) - Converted security.conf from ERB to Jinja2 template. No variables were present in the original template.
- [x] cookbooks/nginx-multisite/templates/default/site.conf.erb → ./ansible/roles/nginx_multisite/templates/site.conf.j2 (complete) - Converted site.conf from ERB to Jinja2 template. Converted ERB variables (@server_name, @document_root, @ssl_enabled, @cert_file, @key_file) to Jinja2 format by removing @ prefix.
- [x] cookbooks/nginx-multisite/templates/default/sysctl-security.conf.erb → ./ansible/roles/nginx_multisite/templates/sysctl-security.conf.j2 (complete) - Converted sysctl-security.conf from ERB to Jinja2 template. No variables were present in the original template.

### Recipes → Tasks
- [x] cookbooks/nginx-multisite/recipes/default.rb → ./ansible/roles/nginx_multisite/tasks/main.yml (complete) - Created main.yml task file that imports the other task files in the correct order. Warnings about non-FQCN are expected since the imported files don't exist yet.
- [x] cookbooks/nginx-multisite/recipes/nginx.rb → ./ansible/roles/nginx_multisite/tasks/nginx.yml (complete) - Created nginx.yml task file that installs nginx, configures it, and deploys static files.
- [x] cookbooks/nginx-multisite/recipes/security.rb → ./ansible/roles/nginx_multisite/tasks/security.yml (complete) - Created security.yml task file that configures fail2ban, UFW firewall, system security settings, and SSH hardening.
- [x] cookbooks/nginx-multisite/recipes/sites.rb → ./ansible/roles/nginx_multisite/tasks/sites.yml (complete) - Created sites.yml task file that configures the Nginx virtual hosts for each site.
- [x] cookbooks/nginx-multisite/recipes/ssl.rb → ./ansible/roles/nginx_multisite/tasks/ssl.yml (complete) - Created ssl.yml task file that installs SSL packages and generates self-signed certificates for each site.

### Attributes → Variables
- [x] cookbooks/nginx-multisite/attributes/default.rb → ./ansible/roles/nginx_multisite/defaults/main.yml (complete) - Created defaults/main.yml with variables converted from Chef attributes.

### Static Files
- [x] cookbooks/nginx-multisite/files/default/test/index.html → ./ansible/roles/nginx_multisite/files/test/index.html (complete) - Copied test/index.html static file.
- [x] cookbooks/nginx-multisite/files/default/ci/index.html → ./ansible/roles/nginx_multisite/files/ci/index.html (complete) - Copied ci/index.html static file.
- [x] cookbooks/nginx-multisite/files/default/status/index.html → ./ansible/roles/nginx_multisite/files/status/index.html (complete) - Copied status/index.html static file.

### Structure Files
- [x] N/A → ./ansible/roles/nginx_multisite/meta/main.yml (complete) - Created meta/main.yml with role metadata.
- [x] N/A → ./ansible/roles/nginx_multisite/handlers/main.yml (complete) - Created handlers/main.yml with handlers for reloading and restarting services.
- [x] N/A → ./ansible/roles/nginx_multisite/tasks/main.yml (complete) - Main tasks file already exists and imports all required task files.
- [x] N/A → ./ansible/roles/nginx_multisite/defaults/main.yml (complete) - Defaults file already exists with all required variables.
- [x] N/A → ansible/roles/nginx_multisite/meta/main.yml (complete)

### Dependencies (requirements.yml)
- [x] collection:community.nginx → ./ansible/roles/nginx_multisite/requirements.yml (complete) - Created requirements.yml with required collections.
- [x] collection:ansible.posix → ./ansible/roles/nginx_multisite/requirements.yml (complete) - Added ansible.posix collection to requirements.yml.
- [x] collection:community.crypto → ./ansible/roles/nginx_multisite/requirements.yml (complete) - Added community.crypto collection to requirements.yml.


Telemetry:
Phase: migrate
Duration: 0.00s

Agent Metrics:
  AAPDiscoveryAgent: 13.27s
    Tokens: 24901 in, 478 out
    Tools: aap_list_collections: 1, aap_search_collections: 2
    collections_found: 0
  PlanningAgent: 89.24s
    Tokens: 267269 in, 4437 out
    Tools: add_checklist_task: 21, list_checklist_tasks: 2, list_directory: 10
  WriteAgent: 400.91s
    Tokens: 584373 in, 6333 out
    Tools: ansible_lint: 6, ansible_write: 8, copy_file: 4, file_search: 2, get_checklist_summary: 1, list_checklist_tasks: 2, read_file: 6, update_checklist_task: 11, write_file: 1
    attempts: 1
    complete: True
    files_created: 22
    files_total: 22
  ValidationAgent: 44.64s
    collections_installed: 2
    collections_failed: 1
    validators_passed: ['ansible-lint', 'role-check']
    validators_failed: []
    attempts: 0
    complete: True
    has_errors: False