# Solution: Advanced Templating with Jinja2

Here are the completed templates and the final playbook for the Jinja exercises.

## Templates

`templates/motd.j2`

```jinja
Welcome to {{ inventory_hostname }}!
Message: {{ custom_message | default('Welcome to the server!') }}
```

`templates/allow_list.conf.j2`

```jinja
{% for ip in allowed_ips %}
allow {{ ip }};
{% endfor %}
deny all;
```

`templates/debug_config.php.j2`

```jinja
{% if enable_debug %}
define( 'WP_DEBUG', true );
{% else %}
define( 'WP_DEBUG', false );
{% endif %}
```

`templates/app.env.j2`

```jinja
{% for key, value in app_env_vars.items() %}
{{ key | upper }}={{ value }}
{% endfor %}
```

## The Playbook

`jinja_exercises.yml`

```yaml
---
- name: Jinja2 Templating Exercises
  hosts: web_server
  become: yes
  vars:
    custom_message: "Ansible is useful"
    allowed_ips:
      - 192.168.1.10
      - 10.0.0.50
      - 127.0.0.1
    enable_debug: true
    app_env_vars:
      db_port: "3306"
      app_env: "production"
      cache_enabled: "true"

  tasks:
    - name: Exercise 1 - Configure MOTD
      ansible.builtin.template:
        src: templates/motd.j2
        dest: /etc/motd

    - name: Exercise 2 - Generate Nginx IP allow list
      ansible.builtin.template:
        src: templates/allow_list.conf.j2
        dest: /tmp/allow_list.conf

    - name: Exercise 3 - Configure WP_DEBUG conditional
      ansible.builtin.template:
        src: templates/debug_config.php.j2
        dest: /tmp/debug_config.php

    - name: Exercise 4 - Generate environment variables
      ansible.builtin.template:
        src: templates/app.env.j2
        dest: /tmp/app.env
```
