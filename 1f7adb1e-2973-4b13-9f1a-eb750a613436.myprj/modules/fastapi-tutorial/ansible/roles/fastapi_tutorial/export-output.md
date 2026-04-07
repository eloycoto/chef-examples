Migration Summary for fastapi_tutorial:
  Total items: 10
  Completed: 10
  Pending: 0
  Missing: 0
  Errors: 0
  Write attempts: 1
  Validation attempts: 0

Final Validation Report:
All migration tasks have been completed successfully

All validations passed

Final checklist:
## Checklist: fastapi_tutorial

### Templates
- [x] N/A → ./ansible/roles/fastapi_tutorial/templates/env.j2 (complete) - Created environment configuration template with Jinja2 variables
- [x] N/A → ./ansible/roles/fastapi_tutorial/templates/fastapi-tutorial.service.j2 (complete) - Created systemd service template with Jinja2 variables

### Recipes → Tasks
- [x] cookbooks/fastapi-tutorial/recipes/default.rb → ./ansible/roles/fastapi_tutorial/tasks/main.yml (complete) - Created tasks file with all necessary tasks from the Chef recipe

### Structure Files
- [x] cookbooks/fastapi-tutorial/metadata.rb → ./ansible/roles/fastapi_tutorial/meta/main.yml (complete) - Created meta/main.yml file with role metadata
- [x] N/A → ./ansible/roles/fastapi_tutorial/tasks/main.yml (complete) - Tasks file already created from Chef recipe
- [x] N/A → ./ansible/roles/fastapi_tutorial/defaults/main.yml (complete) - Created defaults file with all necessary variables
- [x] N/A → ./ansible/roles/fastapi_tutorial/handlers/main.yml (complete) - Created handlers file with systemd reload and service restart handlers
- [x] N/A → ansible/roles/fastapi_tutorial/meta/main.yml (complete)

### Dependencies (requirements.yml)
- [x] collection:community.postgresql → ./ansible/roles/fastapi_tutorial/requirements.yml (complete) - Added community.postgresql collection to requirements.yml
- [x] collection:ansible.posix → ./ansible/roles/fastapi_tutorial/requirements.yml (complete) - Added ansible.posix collection to requirements.yml


Telemetry:
Phase: migrate
Duration: 0.00s

Agent Metrics:
  AAPDiscoveryAgent: 13.78s
    Tokens: 18742 in, 493 out
    Tools: aap_list_collections: 1, aap_search_collections: 2
    collections_found: 0
  PlanningAgent: 51.35s
    Tokens: 105899 in, 2395 out
    Tools: add_checklist_task: 10, get_checklist_summary: 1, list_checklist_tasks: 2, list_directory: 2, read_file: 2
  WriteAgent: 110.38s
    Tokens: 288692 in, 5463 out
    Tools: ansible_lint: 1, ansible_write: 6, list_checklist_tasks: 3, read_file: 2, update_checklist_task: 9, write_file: 2
    attempts: 1
    complete: True
    files_created: 10
    files_total: 10
  ValidationAgent: 18.58s
    collections_installed: 2
    collections_failed: 0
    validators_passed: ['ansible-lint', 'role-check']
    validators_failed: []
    attempts: 0
    complete: True
    has_errors: False