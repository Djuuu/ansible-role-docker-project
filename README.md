Ansible Role: Docker-project
============================

Base role for Docker Compose projects.

Requirements
------------

- Docker CLI with Docker compose plugin 2.18.0 or later
- Ansible >=2.11.0

Role Variables
--------------

Common Docker projects variables:

### Project name
```yaml
docker_project_name: # example-project
```

Project name should be set in concrete roles using this role as a base.

This variable is derived into the following 2 variables:
- `docker_project_slug` _(slugified project name, with `-` separators)_  
  Determines:  
  - directory where project will be installed
  - default service subdomain
  - misc. Traefik keys (service & router, middleware prefix)  
- `docker_project_prefix` _(slugified project name, with `_` separators)_  
  Used as [dynamic variables](#dynamic-variables) prefix  

Implementation example:
* `example-role/defaults/main.yml`
  ```yaml
  example_role_project_name: example
  ```
* `example-role/vars/main.yml`
  ```yaml
  docker_project_name: "{{ example_role_project_name }}"
  ```

### Base directory for Docker projects
```yaml
docker_projects_path: # /var/apps
```

### Base domain suffixes
```yaml
# Local machine domain
local_base_domain: internal

# Exposed service domain
exposed_base_domain: # example.net
```
These domains suffixes are used to build Traefik default rule in
[djuuu.traefik_docker](https://github.com/Djuuu/ansible-role-traefik-docker),
along with optional `{{ docker_project_prefix }}_traefik_subdomain` [dynamic variable](#dynamic-variables). 

See:
- https://github.com/Djuuu/ansible-role-traefik-docker/blob/main/templates/docker-compose.yml.j2#L56-L60
- https://doc.traefik.io/traefik/providers/docker/#defaultrule

### Additional allowed origins
```yaml
docker_project_allowed_origins: []
#  - https://dashboard.example.net
```
Used in `allow-frames@file` and `{{ docker_project_slug }}-cors@docker` middlewares from
[djuuu.traefik_docker](https://github.com/Djuuu/ansible-role-traefik-docker) role.


### Dynamic variables

Variables constructed from `docker_project_name` value can be used to configure: 

* Traefik integration (see [djuuu.traefik_docker](https://github.com/Djuuu/ansible-role-traefik-docker) role)

  - _`{{ docker_project_prefix }}`_**`_traefik_enable`**  
    Should Traefik expose service? (default: `false`)  

  - _`{{ docker_project_prefix }}`_**`_traefik_domain`**  
    Domain used to expose service (note: takes precedence over `_traefik_subdomain`)  

  - _`{{ docker_project_prefix }}`_**`_traefik_subdomain`**  
    Subdomain used to expose service (concatenated with `local_base_domain` / `exposed_base_domain`).  

  - _`{{ docker_project_prefix }}`_**`_traefik_router_service`**  
    Service name targeted by Traefik router  

  - _`{{ docker_project_prefix }}`_**`_traefik_loadbalancer_server_port`**  
    Port targeted by Traefik router  

  - _`{{ docker_project_prefix }}`_**`_traefik_entrypoints`** (default: `"https"`)  
    Traefik router entrypoints  

  - _`{{ docker_project_prefix }}`_**`_traefik_middlewares`**  
    Traefik router middlewares  

  - _`{{ docker_project_prefix }}`_**`_traefik_define_internal_route`** (default: `false`)  
    Should an internal route be defined (restricted to `traefik_internal_networks`)

  - _`{{ docker_project_prefix }}`_**`_traefik_middlewares_internal`**  
    Traefik router middlewares applied on internal route

  - _`{{ docker_project_prefix }}`_**`_access_control_max_age`** (default: `100`)  
  - _`{{ docker_project_prefix }}`_**`_access_control_allow_methods`** (default: `"*"`)  
  - _`{{ docker_project_prefix }}`_**`_access_control_allow_headers`** (default: `"*"`)  
    CORS settings when `{{ docker_project_slug }}-cors@docker` middleware is enabled  

  - _`{{ docker_project_prefix }}`_**`_traefik_custom_labels`**  
    Additional Traefik labels  

* Network subnets

  - _`{{ docker_project_prefix }}`_**`_compose_ipv4_subnet`**  
  - _`{{ docker_project_prefix }}`_**`_compose_ipv6_subnet`**  
    Optional default network IP subnets  

* Additional docker compose options

  - _`{{ docker_project_prefix }}`_**`_compose_additional_options`**  
    Top-level additional docker-compose options
  - _`{{ docker_project_prefix }}`_**`_compose_service_additional_options`**  
    Main service additional docker-compose options (ex: cpu_shares, deploy, ...)

When set, dynamic variables are set into equivalent variables prefixed with `docker_project_`  
(ex: `docker_project_traefik_domain`, `docker_project_compose_additional_options`, etc.)

### Generated variables

These variables are computed from other variables and can be used in roles:

- `docker_project_path`=`"{{ docker_projects_path }}/{{ docker_project_slug }}"`  
  Target directory of current project installation.  
  Should be used in depending role.  

- `traefik_domains`  
  list of fully qualified domains used to expose service (internal use)  
  ex: `['hello-world.example.net', 'hello-world.local']`  

- `traefik_domain_label`  
  Traefik routing rule label when using domain configuration (internal use)  
  ex: ``traefik.http.routers.hello-world.rule: Host(`hello-world.example.net`)``  

- `traefik_subdomain_label`  
  Traefik routing rule label when using subdomain configuration (internal use)  
  ex: `net.example.subdomain: hello-world`  

- **`traefik_project_base_host_rules`**  
  Host rules used in Traefik routing rules.  
  Can be useful when adding custom routing rules (`{{ docker_project_prefix }}_traefik_custom_labels`)  
  ex: ``(Host(`hello-world.example.net`) || Host(`hello-world.local`))``  

- **`traefik_project_base_labels`**  
  Generated Traefik labels.  
  Should be added to the `labels` section of your service Docker Compose configuration.  
  ex:  
  ```yaml  
  traefik.enable: true  
  net.example.subdomain: hello-world  
  # ...  
  ```  

- **`docker_project_default_network`**  
  Generated default network configuration.  
  Should be added to the `networks` section of your Docker Compose configuration.  
  ex:  
  ```yaml  
  default:  
    driver: bridge  
    ipam:  
      driver: default  
      config:  
        - subnet: 172.18.0.0/16  
        - subnet: 2001:db8:0001:0001::/64  
    enable_ipv6: true  
  ```  

### Other variables

```yaml
# Path to the Docker CLI (Community.Docker modules argument)
# https://docs.ansible.com/ansible/latest/collections/community/docker/index.html
docker_cli:
```

`docker_cmd` is available for convenience.  
It uses `docker_cli` if set, or defaults to 'docker'.

Example role implementation
---------------------------

Role example:

* `hello_world/meta/main.yml`
  ```yaml
  galaxy_info:
    role_name: hello_world
  
  dependencies:
    - name: djuuu.docker_project
      src: https://github.com/Djuuu/ansible-role-docker-project
      version: main
  ```
* `hello_world/defaults/main.yml`
  ```yaml
  hello_world_project_name: hello-world
  ```
* `my-example/vars/main.yml`
  ```yaml
  docker_project_name: "{{ hello_world_project_name }}"
  ```
* `hello_world/tasks/main.yml`
  ```yaml
  ## Init
  - name: Ensure project directory exists
    ansible.builtin.file:
      path: "{{ docker_project_path }}"
      state: directory
      mode: '0755'
    tags: ["init"]
  
  ## Configure
  - name: Template docker-compose.yml
    ansible.builtin.template:
      src: docker-compose.yml.j2
      dest: "{{ docker_project_path }}/docker-compose.yml"
      mode: '0644'
      validate: "{{ docker_cmd }} compose -f %s config"
      backup: true
  
  ## Run
  - name: Start app
    ansible.builtin.include_role: { name: djuuu.docker_project, tasks_from: compose-up }
  ```
* `hello_world/templates/docker-compose.yml.j2`
  ```yaml
  name: {{ docker_project_slug }}

  services:

    hello_world:
      image: hello-world

      {{ docker_project_compose_service_additional_options | indent(4) }}

      labels:
        {{ traefik_project_base_labels | indent(6) }}

  networks:
    {{ docker_project_default_network | indent(2) }}

  {{ docker_project_compose_additional_options }}
  ```

Configuration example with dynamic vars:

* `host_vars/example/hello.yml`
```yaml
hello_world_project_name: "hey-there"

# Dynamic variables

hey_there_traefik_enable: true

hey_there_traefik_domain: example.net 
hey_there_traefik_subdomain: hey

hey_there_traefik_router_service: hello 
hey_there_traefik_loadbalancer_server_port: 1234 
hey_there_traefik_entrypoints: http,https
hey_there_traefik_middlewares: 
  - "internal-access@file"
  - "allow-frames@file"
  - "{{ docker_project_slug }}-cors@docker"
    
hey_there_access_control_max_age:       60
hey_there_access_control_allow_methods: "OPTIONS, HEAD, GET"
hey_there_access_control_allow_headers: "X-Custom-Header, Upgrade-Insecure-Requests"
  
hey_there_traefik_custom_labels: |-
  hello-label: hey there
  other-label: example

hey_there_compose_ipv4_subnet: 172.18.0.0/16
hey_there_compose_ipv6_subnet: 2001:db8:0001:0001::/64

hey_there_compose_service_additional_options: |
  cpu_shares: 10
  deploy:
    resources:
      limits:
        memory: 128M
  ports:
    - "1234:1234"

hey_there_compose_additional_options: |
  x-example:
    title: This is an example
```

Dependencies
------------

None.

Some variables allow integration with:
- [djuuu.traefik_docker](https://github.com/Djuuu/ansible-role-traefik-docker)

Example Playbooks
-----------------

Role is not intended to be run as-is, but as a dependency in other roles.

However, there are some available tasks that can be used independently:

**`compose-ps`**
```yaml
- name: List Docker Compose projects status
  hosts: all
  gather_facts: false

  vars:
    docker_projects:
      - my-project
      - other-project
    
  tasks:
    - name: Docker compose ps
      ansible.builtin.include_role:
        name: djuuu.docker_project
        tasks_from: compose-ps
      vars:
        docker_project_name: "{{ item.name | default(item) }}"
        project_restart: "{{ item.restart | default(true) }}"
      loop: "{{ docker_projects | default([]) }}"
```

**`compose-update`**
```yaml
- name: Update Docker Compose projects
  hosts: all
  gather_facts: false

  vars:
    docker_projects:
      - "my-project"
      - { name: "other-project", restart: false } # on-demand service without running container

  tasks:

    - name: Docker compose update
      ansible.builtin.include_role:
        name: djuuu.docker_project
        tasks_from: compose-update
      vars:
        docker_project_name: "{{ item.name | default(item) }}"
        project_restart: "{{ item.restart | default(true) }}"
      loop: "{{ docker_projects | default([]) }}"

    - name: Docker system prune
      ansible.builtin.include_role:
        name: djuuu.docker_project
        tasks_from: system-prune
```

**`compose-up`**
```yaml
- name: Start Docker Compose project
  hosts: example
  gather_facts: false
  vars:
    docker_project_name: my-project
  tasks:
    - name: Start app
      ansible.builtin.include_role:
        name: djuuu.docker_project
        tasks_from: compose-up
```

**`compose-restart`**
```yaml
- name: Restart Docker Compose project
  hosts: example
  gather_facts: false
  vars:
    docker_project_name: my-project
  tasks:
    - name: Restart app
      ansible.builtin.include_role: 
        name: djuuu.docker_project
        tasks_from: compose-restart
```

**`system-prune`**
```yaml
- name: Docker system prune
  hosts: all
  gather_facts: false
  tasks:
    - name: Docker system prune
      ansible.builtin.include_role:
        name: djuuu.docker_project
        tasks_from: system-prune
```

License
-------

Beerware License
