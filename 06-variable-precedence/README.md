# Variables and setting precedence

## Creating valid variable names

| Valid variable names | Not valid                                      |
|----------------------|------------------------------------------------|
| foo                  | *foo, Python keywords such as async and lambda |
| foo_env              | playbook keywords such as environment          |
| foo_port             | foo-port, foo port, foo.port                   |
| foo5, _foo           | 5foo, 12                                       |

## Defining and referencing variables

### Simple Variables

You can define a simple variable using standard YAML syntax. For example:

```yaml 
remote_install_path: /opt/my_app_config
```

To then call the variable within your playbook:

```yaml
ansible.builtin.template:
  src: foo.cfg.j2
  dest: '{{ remote_install_path }}/foo.cfg'
```

**Note on Jinja2:** Notice the `{{ }}` syntax around the variable name. This is Jinja2 templating syntax. It tells Ansible to evaluate the variable inside the brackets instead of treating it as literal text. Jinja templating will be encountered frequently when working with Ansible, like when generating encrypted passwords or managing configurations.

### Boolean Variables

Ansible accepts a broad range of values for boolean variables: `true/false`, `1/0`, `yes/no`, `True/False` and so on. The matching of valid strings is case insensitive. While documentation examples focus on `true/false` to be compatible with `ansible-lint` default settings, you can use any of the following:

| Valid values                                               | Description   |
|------------------------------------------------------------|---------------|
| True , 'true' , 't' , 'yes' , 'y' , 'on' , '1' , 1 , 1.0   | Truthy values |
| False , 'false' , 'f' , 'no' , 'n' , 'off' , '0' , 0 , 0.0 | Falsy values  |

**Warning:** Because words like `on` or `yes` are evaluated as booleans, you must be careful when using them as strings. For example, if you are setting a variable for the province of Ontario to `on`, you must quote it like `province: "on"`, otherwise Ansible will interpret it as `True`.

This flexibility also means that for parameters requiring a boolean, like `become`, you can write `become: yes`, `become: true`, or `become: on` and they will all work exactly the same.

### List Variables

A list variable combines a variable name with multiple values. The multiple values can be stored as an itemized list or within square brackets [], separated with commas.

You can define variables with multiple values using YAML lists. For example:

```yaml
region:
  - northeast
  - southeast
  - midwest
```

When you use variables defined as a list (also called an array), you can access individual items from that list using an index. Ansible (like Python) uses zero-indexed lists, meaning the first item in a list is `[0]`, the second item is `[1]`, and so on. For example, to get the first region from the list above:

```yaml
region_choice: "{{ region[0] }}"
```

### Key:Value Dictionaries

Dictionaries allow you to group related key-value pairs together under a single parent key using indentation.

Define using the format:

```yaml
foo:
  field1: one
  field2: two
```

Reference key:value dictionary with bracket or dot notation:

```yaml
foo['field1']
foo.field1
```

### Registering Variables

You can create variables from the output of an Ansible task with the task keyword `register`. You can use registered variables in any later tasks in your play. For example, the shell command will only be run when the return code (`rc`) of the registered variable equals 5:

```yaml
- hosts: web_servers

  tasks:

     - name: Run a shell command and register its output as a variable
       ansible.builtin.shell: /usr/bin/foo
       register: foo_result
       ignore_errors: true

     - name: Run a shell command using output of the previous task
       ansible.builtin.shell: /usr/bin/bar
       when: foo_result.rc == 5
```

## What is Variable Precedence

Ansible does apply variable precedence, and you might have a use for it. Variable precedence can be useful to set baselines for a typical deployment while allowing for more granular control when needed. In general, Ansible ***gives precedence to variables that were defined more recently, more actively, and with more explicit scope***. 

**Best Practice:** As an organization, it is highly important to decide on a standard for where your variables will go. Because Ansible gives you so many different places to define variables, things can get extremely confusing if you don't have a clear strategy.

While the list below is long, for now, you only need to understand the most important ones:

- **Role Defaults (Level 2):** These really are just *defaults*. They have almost the lowest precedence and are easily overridden.
- **Task/Role Vars (Levels 15-17):** Variables defined directly on a task or within a role's `vars` directory have high precedence and will override most other things.
- **Extra Vars (Level 22):** Variables passed at the command line using `-e` or `--extra-vars` will **always** win precedence.

Here is the order of precedence from least to greatest (the last listed variables override all other variables):

1. command line values (for example, -u my_user, these are not variables)
2. role defaults (as defined in Role directory structure)
3. inventory file or script group vars 
4. inventory group_vars/all
5. playbook group_vars/all 
6. inventory group_vars/* 
7. playbook group_vars/* 
8. inventory file or script host vars 
9. inventory host_vars/* 
10. playbook host_vars/* 
11. host facts / cached set_facts 
12. play vars
13. play vars_prompt
14. play vars_files
15. role vars (as defined in Role directory structure)
16. block vars (only for tasks in block)
17. task vars (only for the task)
18. include_vars
19. set_facts / registered vars
20. role (and include_role) params
21. include params
22. extra vars (for example, -e "user=my_user") (always win precedence)

## Scoping Variables

You can decide where to set a variable based on the scope you want that value to have. Ansible has three main scopes:

- Global: this is set by config, environment variables and the command line
- Play: each play and contained structures, vars entries (vars; vars_files; vars_prompt), role defaults and vars.
- Host: variables directly associated to a host, like inventory, include_vars, facts or registered task outputs

Inside a template, you automatically have access to all variables that are in scope for a host, plus any registered variables, facts, and magic variables.

## Examples of scoping and replacing variables

Set common defaults in a `group_vars/all` file

```yaml
---
# file: /etc/ansible/group_vars/all
# this is the site wide default
ntp_server: default-time.example.com
```

Set location-specific variables in `group_vars/my_location` files. All groups are children of the `all` group, so variables set here override those set in `group_vars/all`:

```yaml
---
# file: /etc/ansible/group_vars/boston
ntp_server: boston-time.example.com
```

If one host used a different NTP server, you could set that in a host_vars file, which would override the group variable:

```yaml
---
# file: /etc/ansible/host_vars/xyz.boston.example.com
ntp_server: override.example.com
```

Set defaults in roles to avoid undefined-variable errors. If you share your roles, other users can rely on the reasonable defaults you added in the `roles/x/defaults/main.yml`file, or they can easily override those values in inventory or at the command line.

```yaml
---
# file: roles/x/defaults/main.yml
# if no other value is supplied in inventory or as a parameter, this value will be used
http_port: 80
```

Set variables in roles to ensure a value is used in that role, and is not overridden by inventory variables. If you are not sharing your role with others, you can define app-specific behaviors like ports this way, in `roles/x/vars/main.yml`. If you are sharing roles with others, putting variables here makes them harder to override, although they still can by passing a parameter to the role or setting a variable with `-e`:

```yaml
---
# file: roles/x/vars/main.yml
# this will absolutely be used in this role
http_port: 80
```

Pass variables as parameters when you call roles for maximum clarity, flexibility, and visibility. This approach overrides any defaults that exist for a role. For example:

```yaml
roles:
   - role: apache
     vars:
        http_port: 8080
```

Instead of worrying about variable precedence, we encourage you to think about how easily or how often you want to override a variable when deciding where to set it. If you are not sure what other variables are defined, and you need a particular value, use `--extra-vars (-e)` to override all other variables.
