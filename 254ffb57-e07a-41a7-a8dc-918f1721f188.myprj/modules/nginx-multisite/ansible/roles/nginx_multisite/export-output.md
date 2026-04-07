Migration Summary for nginx_multisite:
  Total items: 20
  Completed: 20
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
- [x] cookbooks/nginx-multisite/templates/default/fail2ban.jail.local.erb → ./ansible/roles/nginx_multisite/templates/fail2ban.jail.local.j2 (complete) - Converted fail2ban jail configuration from ERB to Jinja2 template
- [x] cookbooks/nginx-multisite/templates/default/nginx.conf.erb → ./ansible/roles/nginx_multisite/templates/nginx.conf.j2 (complete) - Converted nginx.conf from ERB to Jinja2 template
- [x] cookbooks/nginx-multisite/templates/default/security.conf.erb → ./ansible/roles/nginx_multisite/templates/security.conf.j2 (complete) - Converted security.conf from ERB to Jinja2 template
- [x] cookbooks/nginx-multisite/templates/default/site.conf.erb → ./ansible/roles/nginx_multisite/templates/site.conf.j2 (complete) - Converted site.conf from ERB to Jinja2 template, replacing ERB variables with Jinja2 syntax
- [x] cookbooks/nginx-multisite/templates/default/sysctl-security.conf.erb → ./ansible/roles/nginx_multisite/templates/sysctl-security.conf.j2 (complete) - Converted sysctl-security.conf from ERB to Jinja2 template

### Recipes → Tasks
- [x] cookbooks/nginx-multisite/recipes/default.rb → ./ansible/roles/nginx_multisite/tasks/main.yml (complete) - Converted default.rb recipe to main.yml tasks with import_tasks for all required task files
- [x] cookbooks/nginx-multisite/recipes/nginx.rb → ./ansible/roles/nginx_multisite/tasks/nginx.yml (complete) - Converted nginx.rb recipe to nginx.yml tasks with package, template, service, file, and copy modules
- [x] cookbooks/nginx-multisite/recipes/security.rb → ./ansible/roles/nginx_multisite/tasks/security.yml (complete) - Converted security.rb recipe to security.yml tasks with package, service, template, command, and lineinfile modules
- [x] cookbooks/nginx-multisite/recipes/sites.rb → ./ansible/roles/nginx_multisite/tasks/sites.yml (complete) - Converted sites.rb recipe to sites.yml tasks with template and file modules
- [x] cookbooks/nginx-multisite/recipes/ssl.rb → ./ansible/roles/nginx_multisite/tasks/ssl.yml (complete) - Converted ssl.rb recipe to ssl.yml tasks with package, group, file, and shell modules

### Attributes → Variables
- [x] cookbooks/nginx-multisite/attributes/default.rb → ./ansible/roles/nginx_multisite/defaults/main.yml (complete) - Converted attributes/default.rb to defaults/main.yml with Ansible variable format

### Static Files
- [x] cookbooks/nginx-multisite/files/default/test/index.html → ./ansible/roles/nginx_multisite/files/test/index.html (complete) - Copied test/index.html static file
- [x] cookbooks/nginx-multisite/files/default/ci/index.html → ./ansible/roles/nginx_multisite/files/ci/index.html (complete) - Copied ci/index.html static file
- [x] cookbooks/nginx-multisite/files/default/status/index.html → ./ansible/roles/nginx_multisite/files/status/index.html (complete) - Copied status/index.html static file

### Structure Files
- [x] N/A → ./ansible/roles/nginx_multisite/meta/main.yml (complete) - Already completed - meta/main.yml was created based on metadata.rb
- [x] N/A → ./ansible/roles/nginx_multisite/defaults/main.yml (complete) - Already completed - defaults/main.yml was created when converting attributes/default.rb
- [x] N/A → ./ansible/roles/nginx_multisite/handlers/main.yml (complete) - Created handlers/main.yml with handlers for nginx, fail2ban, ssh, and sysctl
- [x] N/A → ./ansible/roles/nginx_multisite/tasks/main.yml (complete) - Already completed - tasks/main.yml was created when converting recipes/default.rb
- [x] cookbooks/nginx-multisite/metadata.rb → ./ansible/roles/nginx_multisite/meta/main.yml (complete) - Created meta/main.yml with role metadata based on Chef metadata.rb
- [x] N/A → ansible/roles/nginx_multisite/meta/main.yml (complete)


Telemetry:
Phase: migrate
Duration: 0.00s

Agent Metrics:
  AAPDiscoveryAgent: 13.10s
    Tokens: 23613 in, 451 out
    Tools: aap_list_collections: 1, aap_search_collections: 2
    collections_found: 0
  PlanningAgent: 84.14s
    Tokens: 265723 in, 4203 out
    Tools: add_checklist_task: 19, list_checklist_tasks: 2, list_directory: 10, read_file: 1
  WriteAgent: 301.27s
    Tokens: 397726 in, 4362 out
    Tools: ansible_doc_lookup: 1, ansible_lint: 5, ansible_write: 6, copy_file: 3, get_checklist_summary: 1, list_checklist_tasks: 1, read_file: 1, update_checklist_task: 8
    attempts: 1
    complete: True
    files_created: 20
    files_total: 20
  ValidationAgent: 5.29s
    validators_passed: ['ansible-lint', 'role-check']
    validators_failed: []
    attempts: 0
    complete: True
    has_errors: False