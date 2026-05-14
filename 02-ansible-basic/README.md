# Ansible Basics

Create your first 'playbook'. 

Create and edit `first-playbook.yml` with the following content. 

```yaml
---
- hosts: all
  tasks:
    - name: echo
      command: echo "First task"
```

- `hosts`: Selects the group of hosts to run the playbook on. In this case, `all` selects all hosts in the inventory.
- `tasks`: Defines the tasks to run on the hosts. In this case, there is only one task.
- `name`: Defines the name of the task. 
- `command`: Runs a command on the hosts. 

Run the playbook.

```bash
ansible-playbook first-playbook.yml
```

So we can see that it ran and did something, but not what it actually did. There a few options to see that. The first would be to use verbose mode, using the `-v` flag. Adding more `v`'s adds more verbosity. 

```bash
ansible-playbook -v first-playbook.yml
```

The other way is to use the `debug` task. Before we can use that we need to capture the output of the first command. 

Edit `first-playbook.yml` with the following content. 

```yaml
---
- hosts: all
  tasks:
    - name: echo
      command: echo "First task"
      register: echo

    - name: debug var echo
      debug:
       var: echo

    - name: debug msg echo
      debug:
        msg: "{{ echo }}"
```

- `register`: Registers the output of the task to a variable. This is useful for capturing the output of a task and using it in a subsequent task. 
- `debug`: Displays the output of a task. 
- `var: echo`: Displays the contents of the variable `echo`. 
- `msg: "{{ echo }}"`: Displaying the variable within a message string. `{{ }}` is the Jinja2 templating syntax for variables. The content of the `{{ }}` block is evaluated with syntax similar to that of Python. 

Note that the `debug var echo` and `debug msg echo` tasks are very similar, but the `msg` parameter is generally the most useful for displaying variables within a message string. 

See the [Ansible Debug Module Documentation](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/debug_module.html) for more information.

- It is normally most useful to start by looking at the `Examples` section of the documentation. 
- Then, if more info is required, look at the `Parameters` section to find the correct parameter to use.
- **Note**: It is normally easiest to search for the module you want to use on Google and follow the link to the official Ansible documentation, since the search within the Ansible documentation itself isn't the best.

Run the playbook again. We can now see the entire debug output for each task. 

```bash
ansible-playbook first-playbook.yml
```

The first part of the output is shown below. 

```text
ok: [webserver] => {
    "msg": {
        "changed": true,
        "cmd": [
            "echo",
            "First",
            "task"
        ],
        "delta": "0:00:00.006185",
        "end": "2026-05-14 15:34:12.365063",
        "failed": false,
        "msg": "",
        "rc": 0,
        "start": "2026-05-14 15:34:12.358878",
        "stderr": "",
        "stderr_lines": [],
        "stdout": "First task",
        "stdout_lines": [
            "First task"
        ]
    }
}
```

This gives similar output to when the `-v` option was used, but we now have much more control over the specific output.

- `msg`: The variable that is being displayed. 
- `cmd`: The actual command that was run, in this case `echo "First task"`.
- `rc`: The return code of the command. In this case `0` indicates that the command was successful.
- `stdout`: The standard output of the command, in our case, `First Task`. 

This playbook is very static and won't be too useful in a normal environment. We'll add variables to make it more dynamic.

We want to use the `command` module, but passing in a variable to the echo command.

Open the [Ansible Command Module Documentation](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/command_module.html), and under the Example section, find a similar example. 

Update `first-playbook.yml` to match the following. 

```yaml
---
- hosts: all
  vars:
    msg: variable
  tasks:
    - name: echo
      command: 'echo "{{ msg }}"'
```

The `vars` block defines variables for only this playbook. We now echo the value of the `msg` variable instead of hard coding a message. We can now modify the output without having to modify the tasks themselves. 
 
Run the playbook, and check that the output is what you expect. 

## YAML Overview

Before continuing, take a moment to familiarize yourself with YAML. The `first-playbook.yml` file is written in YAML format. You can read more about YAML from the [official YAML documentation](https://yaml.org/).

YAML is a human-readable data serialization format that is often used for configuration files, and data exchange between languages. 

### Key Concepts of YAML for Ansible

**1. Key-Value Pairs (Dictionaries)**
YAML relies heavily on key-value pairs separated by a colon and a space.

```yaml
name: "Ubuntu Server"
ip_address: 192.168.1.10
```

**2. Lists (Arrays)**
Lists are represented by a hyphen and a space (`-`) before each item. You will see this often when defining tasks or lists of packages to install.

```yaml
packages:
  - nginx
  - postgresql
  - python3
```

In the example above, `packages` is a parent key that contains a list of three strings. 

**3. Indentation**
YAML uses indentation to represent structure and nesting. **You must use spaces, not tabs.** Typically, two spaces are used per indentation level.

```yaml
server:
  hostname: web01
  interfaces:
    - eth0
    - eth1
```

**4. Booleans**
True/False values are often used in Ansible to enable or disable settings. They can be written as `true`/`false`, `yes`/`no`.

```yaml
create_user: yes
enable_ssh: true
```

**5. Dictionaries (Maps/Hashes)**
Dictionaries allow you to group related key-value pairs together under a single parent key using indentation. This is very common for defining complex variables.

```yaml
user_info:
  username: "john"
  shell: "/bin/bash"
  groups:
    - sudo
    - admin
```

In the above example, `user_info` is a parent key that contains three key-value pairs, `username`, `shell`, and `groups`. The `groups` key is a list of strings. 

## Lists in Ansible

Ansible has the concept of lists, and items of a list can be looped through. In older versions of ansible you will see this referenced as `with_items`. It is still backwards compatible but will be deprecated eventually.

Update `first-playbook.yml` to match the following. 

```yaml
---
- hosts: all
  vars:
    msg:
      - one
      - two
      - three
  tasks:
    - name: echo with var
      command: 'echo "{{ item }}"'
      loop: "{{ msg }}"

    - name: echo with list
      command: 'echo "{{ item }}"'
      loop:
        - four
        - five
        - six
```

- `echo with var`: This task loops through the items in the `msg` list and prints them to the console.
- `echo with list`: This task loops through a list of strings and prints them to the console, without having them saved in a variable first.

Run the playbook to see its output. 

This playbook works, but is not "best practice" for the current versions of ansible. We're referring to the modules (like `command`, `debug`, ...) only by their short names, and not their collection. By default, ansible will assume you are referring to the `ansible.builtin` collection, but if a local module has the same name, then it will be used instead and could cause issues. 

Rewrite the playbook to use the full module names. 

`first-playbook.yml`

```yaml
- hosts: all
  vars:
    msg:
      - one
      - two
      - three
  tasks:
    - name: echo with var
      ansible.builtin.command:
        cmd: 'echo "{{ item }}"'
      register: echo
      loop: "{{ msg }}"

    - name: debug var echo
      ansible.builtin.debug:
       var: echo

    - name: echo with list
      ansible.builtin.command: 'echo "{{ item }}"'
      register: echo
      loop:
        - four
        - five
        - six

    - name: debug var echo
      ansible.builtin.debug:
       var: echo
```

While using these full module names takes longer to type, it is more explicit and reduces the risk of using the wrong module, as well as making it much more secure in a production environment. 

Run the playbook to make sure that the functionality has not changed. 

Lastly, move the variable definitions to the inventory file, as they make more sense as global variables.

Update the following 2 files, then run the playbook again. 

`inventory/hosts.yml`

```yaml
nginx:
  hosts:
    webserver
ubuntu:
  hosts:
    webserver:
      ansible_host: webserver
      msg:
        - one
        - two
        - three
    pki-server:
      ansible_host: pki-server
      msg:
        - four
        - five
        - six
  vars:
    ansible_user: ansibleuser
```

While there are many places to store variables, the general rule is to keep variables as close to where they are used as possible. In this case, the `msg` variable is used by both the `webserver` and `pki-server` hosts, so it makes sense to store it in the inventory file. 

`first-playbook.yml`

```yaml
- hosts: all
  tasks:
    - name: echo with var
      ansible.builtin.command:
        cmd: 'echo "{{ item }}"'
      register: echo
      loop: "{{ msg }}"

    - name: debug var echo
      ansible.builtin.debug:
       var: echo
```

- `item`: This is the current item in the loop, as the list is iterated through. In this case, on `webserver`, `item` will be `one`, then `two`, then `three`.

**Note:** If you look at older Ansible tutorials or codebases, you will likely come across `with_items` (or other `with_*` variants) instead of `loop`. While these still work for backwards compatibility, they are considered legacy syntax. Using `loop` is the modern, recommended best practice for iterating over lists in Ansible.
