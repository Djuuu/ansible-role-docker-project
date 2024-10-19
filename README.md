Docker-tasks
============

Common Docker tasks.


Role Variables
--------------

Common Docker projects variables:

```yaml
# Base directory for Docker projects
docker_projects_path: /var/www
```

Dependencies
------------

None.

Example Playbook
----------------

```yaml
- hosts: all
  
  vars:
    project_name: my-project
    project_path: "/var/www/{{ project_name }}"
    
  tasks:
    - name: Init Docker Compose project vars
      ansible.builtin.include_role: { name: djuuu.docker_tasks, tasks_from: init-compose-project-vars }
      
    # ...
    
    - name: Start app
      ansible.builtin.include_role: { name: djuuu.docker_tasks, tasks_from: compose-up }
```

License
-------

Beerware License
