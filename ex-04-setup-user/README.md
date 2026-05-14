# Exercise: Setting up a New User with SSH Access

In previous exercises, we've been using `ansibleuser` to connect to our servers. In a real-world scenario, you will often need to provision new users and configure them with proper access controls (like SSH keys and sudo privileges).

In this exercise, you will create a playbook that provisions a new `deployuser` across both your `webserver` and `pki-server` machines.

## Setup

Create a new playbook named `setup_user.yml`.

`setup_user.yml`

```yaml
---
- name: Provision a new deploy user
  hosts: all
  become: yes
  vars:
    new_user: "deployuser"
    ssh_pub_key: "{{ lookup('file', '~/.ssh/id_ed25519.pub') }}"

  tasks:
```

## The Challenge

Using the Ansible documentation, figure out how to write the 3 tasks required to provision the user.

### Task 1: Create the User

**Exercise:** Using the [Ansible `ansible.builtin.user` documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/user_module.html), write a task to create a user using the `{{ new_user }}` variable. Ensure the user has a home directory and that their default shell is set to `/bin/bash`.

### Task 2: Configure SSH Key Access

**Exercise:** Using the [Ansible `ansible.posix.authorized_key` documentation](https://docs.ansible.com/ansible/latest/collections/ansible/posix/authorized_key_module.html), write a task that takes the `{{ ssh_pub_key }}` variable we defined and adds it to the authorized keys for the newly created `{{ new_user }}`.

### Task 3: Grant Passwordless Sudo Access

**Exercise:** Using the [Ansible `ansible.builtin.copy` documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html), write a task that creates a new file at `/etc/sudoers.d/deployuser`. The contents of the file should literally be: `deployuser ALL=(ALL) NOPASSWD: ALL`. 

## Verification

Once your playbook is complete, run it. 

```bash
ansible-playbook setup_user.yml
```

To verify it worked, try to SSH directly into one of your servers using the new user. It should log you in instantly without asking for a password.

```bash
ssh deployuser@{server IP}
```

And verify you have sudo access without a password:

```bash
sudo ls /root
```
