# Exercise: Advanced Templating with Jinja2

In previous exercises, you've used the `ansible.builtin.template` module to copy configurations (like `wp-config.php.j2` or `default.j2`) to your servers. However, Jinja2 can do much more than simple variable substitution. 

In this exercise, you will create a single playbook `jinja_exercises.yml` that applies 4 different templates, each focusing on a different Jinja2 feature.

## Setup

First, create a directory for your templates if you don't already have one:

```bash
mkdir templates
```

Next, create the `jinja_exercises.yml` playbook. You will add tasks to this file as you complete each exercise.

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
```

---

## Exercise 1: The `default()` Filter

Sometimes, a variable might not be defined. Jinja2 provides filters to handle this gracefully. The `default()` filter allows you to specify a fallback value.

1. Create a template named `templates/motd.j2`.
2. Write a Message of the Day that includes the server's hostname using the built-in `{{ inventory_hostname }}` variable.
3. Also include a custom message using `{{ custom_message }}`. However, apply the `default()` filter so that if `custom_message` isn't defined, it falls back to printing `"Welcome to the server"`.

    Example Template snippet for `templates/motd.j2`:

    ```jinja
    Welcome to {{ inventory_hostname }}
    Message: {{ custom_message | default('Welcome to the server') }}
    ```

4. Add an `ansible.builtin.template` task to your playbook to copy this file to `/etc/motd`.

## Exercise 2: Jinja Loops (`{% for %}`)

You can iterate over lists directly inside your templates to generate repetitive configuration blocks without hardcoding them.

Imagine you need to restrict access to an Nginx server based on a dynamic list of IPs.

1. Create a template named `templates/allow_list.conf.j2`.
2. Use a `{% for ip in allowed_ips %}` loop to generate a list of `allow` statements.
3. End the file with a `deny all;` statement.

    Example Template snippet for `templates/allow_list.conf.j2`:

    ```jinja
    allow 192.168.1.10;
    allow 10.0.0.50;
    allow 127.0.0.1;
    deny all;
    ```

4. Add a task to your playbook to copy this template to `/tmp/allow_list.conf`. (We are using `/tmp/` so we don't accidentally break Nginx).

## Exercise 3: Conditionals (`{% if %}`)

Conditionals allow you to turn entire blocks of configuration on or off based on a boolean variable.

Let's pretend we are modifying `wp-config.php`.

1. Create a template named `templates/debug_config.php.j2`.
2. Use an `{% if enable_debug %}` statement. If true, the file should print `define( 'WP_DEBUG', true );`. 
3. Use an `{% else %}` statement so that if it is false, it prints `define( 'WP_DEBUG', false );`.
4. Make sure to close your conditional with `{% endif %}`. 

5. Add a task to your playbook to copy this template to `/tmp/debug_config.php`. Play around with changing `enable_debug: true` to `false` in your playbook vars to see the output change. 

## Exercise 4: Dictionary Iteration & String Filters

You can iterate over Key-Value pairs (Dictionaries) as well. You can also chain multiple filters together.

We need to generate a `.env` file for a web application based on the `app_env_vars` dictionary. However, the application requires all the keys in the `.env` file to be **UPPERCASE**, even though they are lowercase in our Ansible vars. 

1. Create a template named `templates/app.env.j2`.
2. Iterate over the dictionary using `{% for key, value in app_env_vars.items() %}`.
3. Inside the loop, print the key and value like `KEY=value`.
4. Apply the `upper` filter to the `key` variable to capitalize it dynamically.

*Example Template snippet:*

```jinja
{% for key, value in app_env_vars.items() %}
{{ key | upper }}={{ value }}
{% endfor %}
```

1. Add a task to your playbook to copy this template to `/tmp/app.env`. 

---

## Verify Your Work

Run your playbook. 

```bash
ansible-playbook jinja_exercises.yml
```

Once it finishes, SSH into your target server and inspect the generated files in `/etc/motd` and `/tmp/` to verify that your Jinja2 templating logic worked exactly as expected. 

```bash
cat /etc/motd
cat /tmp/allow_list.conf
cat /tmp/debug_config.php
cat /tmp/app.env
```
