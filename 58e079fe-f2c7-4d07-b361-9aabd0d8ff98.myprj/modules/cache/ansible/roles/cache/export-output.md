Migration Summary for cache:
  Total items: 27
  Completed: 27
  Pending: 0
  Missing: 0
  Errors: 0
  Write attempts: 1
  Validation attempts: 0

Final Validation Report:
All migration tasks have been completed successfully

Validation passed with warnings:
ansible-lint: Passed with 3 warning(s):
[VERY_HIGH] defaults/main.yml:1 [schema] $ [{'include_vars': 'memcached.yml'}, {'include_vars': 'redis.yml'}] is not of type 'object'. See https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_variables.html ( Returned errors will not include exact line numbers, but they will mention
the schema name being used as a tag, like ``schema[playbook]``,
``schema[tasks]``.

This rule is not skippable and stops further processing of the file.

If incorrect schema was picked, you might want to either:

* move the file to standard location, so its file is detected correctly.
* use ``kinds:`` option in linter config to help it pick correct file type.
)
[LOW] tasks/redis_disable_default.yml:1 [ignore-errors] Use failed_when and specify error conditions instead of using ignore_errors. (Task/Handler: Stop and disable default Redis service on Debian/Ubuntu)
[LOW] tasks/redis_disable_default.yml:8 [ignore-errors] Use failed_when and specify error conditions instead of using ignore_errors. (Task/Handler: Stop and disable default Redis service on RedHat/CentOS)

==============================
Rule Hints (How to Fix):
==============================
# schema

Validates Ansible metadata files against JSON schemas.

## Common schema validations

- `schema[playbook]`: Validates playbooks
- `schema[tasks]`: Validates task files in `tasks/**/*.yml`
- `schema[vars]`: Validates variable files in `vars/*.yml` and `defaults/*.yml`
- `schema[meta]`: Validates role metadata in `meta/main.yml`
- `schema[galaxy]`: Validates collection metadata
- `schema[requirements]`: Validates `requirements.yml`

## Problematic code (meta/main.yml)

```yaml
galaxy_info:
  author: example
  # Missing standalone key
```

## Correct code (meta/main.yml)

```yaml
galaxy_info:
  standalone: true # <- Required to clarify role type
  author: example
  description: Example role
```

**Tip:** For `meta/main.yml`, always include `galaxy_info.standalone` property. Empty meta files are not allowed.

# ignore-errors

Use conditional ignoring, register errors, or define specific failure conditions instead of blindly ignoring all errors.

## Problematic code

```yaml
- name: Run apt-get update
  ansible.builtin.command: apt-get update
  ignore_errors: true # Ignores all errors
```

## Correct code

```yaml
# Option 1: Ignore only in check mode
- name: Run apt-get update
  ansible.builtin.command: apt-get update
  ignore_errors: "{{ ansible_check_mode }}"

# Option 2: Register and handle errors
- name: Run apt-get update
  ansible.builtin.command: apt-get update
  ignore_errors: true
  register: update_result

# Option 3: Define specific failure conditions
- name: Disable apport
  lineinfile:
    line: "enabled=0"
    dest: /etc/default/apport
  register: result
  failed_when: result.rc != 0 and result.rc != 257
```

Final checklist:
## Checklist: cache

### Templates
- [x] /workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/templates/default/redis.conf.erb → ./ansible/roles/cache/templates/redis.conf.j2 (complete) - Created Redis configuration template with Jinja2 variables
- [x] /workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/templates/default/redis.init.erb → ./ansible/roles/cache/templates/redis.init.j2 (complete) - Created Redis init script template with Jinja2 variables
- [x] /workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/templates/default/redis.upstart.conf.erb → ./ansible/roles/cache/templates/redis.upstart.conf.j2 (complete) - Created Redis upstart configuration template with Jinja2 variables
- [x] /workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/templates/default/redis@.service.erb → ./ansible/roles/cache/templates/redis@.service.j2 (complete) - Created Redis systemd service template
- [x] cookbooks/memcached/templates/default/memcached.conf.erb → ./ansible/roles/cache/templates/memcached.conf.j2 (complete) - Created Memcached configuration template with Jinja2 variables

### Recipes → Tasks
- [x] cookbooks/cache/recipes/default.rb → ./ansible/roles/cache/tasks/main.yml (complete) - Created main tasks file with import_tasks for memcached and redis
- [x] /workspace/source/migration-dependencies/cookbook_artifacts/memcached-7992788f1a376defb902059063f5295e37d281cb/recipes/default.rb → ./ansible/roles/cache/tasks/memcached.yml (complete) - Created memcached tasks file with package, template, and service tasks
- [x] /workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/recipes/default.rb → ./ansible/roles/cache/tasks/redis.yml (complete) - Created Redis tasks file with import_tasks for Redis components
- [x] /workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/recipes/_install_prereqs.rb → ./ansible/roles/cache/tasks/redis_prereqs.yml (complete) - Created Redis prerequisites tasks file
- [x] /workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/recipes/install.rb → ./ansible/roles/cache/tasks/redis_install.yml (complete) - Created Redis installation tasks file
- [x] /workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/recipes/ulimit.rb → ./ansible/roles/cache/tasks/redis_ulimit.yml (complete) - Created Redis ulimit configuration tasks file
- [x] /workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/recipes/disable_os_default.rb → ./ansible/roles/cache/tasks/redis_disable_default.yml (complete) - Created Redis disable default service tasks file
- [x] /workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/recipes/configure.rb → ./ansible/roles/cache/tasks/redis_configure.yml (complete) - Created Redis configuration tasks file
- [x] /workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/recipes/enable.rb → ./ansible/roles/cache/tasks/redis_enable.yml (complete) - Created Redis enable tasks file
- [x] N/A → ./ansible/roles/cache/tasks/main.yml (complete) - Created main tasks file that includes memcached and redis tasks
- [x] /workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/recipes/install.rb → ./ansible/roles/cache/tasks/redis.yml (complete) - Created Redis installation and configuration tasks
- [x] cookbooks/memcached/recipes/default.rb → ./ansible/roles/cache/tasks/memcached.yml (complete) - Created Memcached installation and configuration tasks

### Attributes → Variables
- [x] /workspace/source/migration-dependencies/cookbook_artifacts/memcached-7992788f1a376defb902059063f5295e37d281cb/attributes/default.rb → ./ansible/roles/cache/defaults/memcached.yml (complete) - Created Memcached default variables file
- [x] /workspace/source/migration-dependencies/cookbook_artifacts/redisio-cac70a2ec9102cac4f5391358c8565d244f5d4db/attributes/default.rb → ./ansible/roles/cache/defaults/redis.yml (complete) - Created Redis default variables file
- [x] cookbooks/memcached/attributes/default.rb → ./ansible/roles/cache/defaults/memcached.yml (complete) - Created Memcached default variables file

### Static Files
- [x] N/A → ./ansible/roles/cache/README.md (complete) - Created README.md with role documentation

### Structure Files
- [x] cookbooks/cache/metadata.rb → ./ansible/roles/cache/meta/main.yml (complete) - Created meta/main.yml file with role metadata
- [x] N/A → ./ansible/roles/cache/handlers/main.yml (complete) - Created handlers file with restart handlers for memcached and redis
- [x] N/A → ./ansible/roles/cache/defaults/main.yml (complete) - Created main defaults file that includes memcached and redis vars
- [x] N/A → ansible/roles/cache/meta/main.yml (complete)

### Dependencies (requirements.yml)
- [x] collection:community.general → ./ansible/roles/cache/requirements.yml (complete) - Created requirements.yml file with community.general collection
- [x] collection:ansible.posix → ./ansible/roles/cache/requirements.yml (complete) - Added ansible.posix collection to requirements.yml


Telemetry:
Phase: migrate
Duration: 0.00s

Agent Metrics:
  AAPDiscoveryAgent: 11.92s
    Tokens: 25054 in, 431 out
    Tools: aap_list_collections: 1, aap_search_collections: 2
    collections_found: 0
  PlanningAgent: 79.41s
    Tokens: 219893 in, 4096 out
    Tools: add_checklist_task: 20, file_search: 2, list_checklist_tasks: 2, list_directory: 2
  WriteAgent: 333.74s
    Tokens: 447656 in, 6676 out
    Tools: add_checklist_task: 8, ansible_write: 10, get_checklist_summary: 1, list_checklist_tasks: 2, update_checklist_task: 7, write_file: 3
    attempts: 1
    complete: True
    files_created: 27
    files_total: 27
  ValidationAgent: 29.72s
    collections_installed: 2
    collections_failed: 0
    validators_passed: ['ansible-lint', 'role-check']
    validators_failed: []
    attempts: 0
    complete: True
    has_errors: False