Docker-project
==============

Base role for Docker Compose projects.

Role Variables
--------------

Common Docker projects variables:

```yaml
# Base directory for Docker projects
docker_projects_path: /var/apps

# Local machine domain
local_base_domain: localhost

# Exposed service domain
exposed_base_domain:
#exposed_base_domain: example.net

# Additional allowed origins
docker_project_allowed_origins: []
#  - https://dashboard.example.net
```

Dependencies
------------

None.

Example Playbook
----------------

```yaml
- hosts: all

  vars:
    docker_project_name: my-project
    docker_projects_path: /var/apps

  tasks:
    - name: Init Docker Compose project vars
      ansible.builtin.include_role: { name: djuuu.docker_project, tasks_from: _init-vars }

    # ...

    - name: Start app
      ansible.builtin.include_role: { name: djuuu.docker_project, tasks_from: compose-up }
```

License
-------

Beerware License
