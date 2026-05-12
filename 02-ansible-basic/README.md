# Ansible Basics

Create your first 'playbook'. 

`first-playbook.yml`

```yaml
---
- hosts: all
  tasks:
    - name: echo
      command: echo "First task"
```

Run the playbook.

```bash
ansible-playbook first-playbook.yml
```

So we can see that it ran and did something, but not what it actually did. There a few options to see that. The first would be to use verbose mode `-v`.

```bash
ansible-playbook -v first-playbook.yml
```

The other way is to use the `debug` task. Before we can use that we need to capture the output of the first command. 

Edit the `first-playbook.yml` file to the following. 

`first-playbook.yml`

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
        msg: "{{echo}}"
```

Run the playbook again. We can now see the entire debug output for each task. 

```bash
ansible-playbook first-playbook.yml
```

This playbook is very static and won't be too useful in a normal environment. We'll add variables to make it more dynamic.

Update `first-playbook.yml` to match the following. 

```yaml
---
- hosts: all
  vars:
    msg: "variable"
  tasks:
    - name: echo
      command: 'echo "{{ msg }}"'
```

Run the playbook. 

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

Run the playbook to see its output. 

This playbook works, but is not "best practice" for the current versions of ansible. We're referring to the modules only by their names and not their collection. By default, ansible will assume you are referring to the `ansible.builtin` collection, but if a local module has the same name, then it will be used and could cause issues. So rewriting the entire playbook with the collection names will look like this.

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
