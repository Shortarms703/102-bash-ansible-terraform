# Installing Docker-ce using Ansible

## Verify Manual Install Steps

Take a look at the manual installation method and see the steps required each time docker needs to be installed.

[https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)

## Creating the Docker Role

Instead of a standard playbook, we will build this as a reusable role. 

First, initialize the new role:

```bash
cd roles
ansible-galaxy init docker
```

All the tasks below should be placed into your `roles/docker/tasks/main.yml` file.

## Automated with Ansible

Talk through the following tasks and see how the requirements from the manual install have been recreated in Ansible. 

**Note**: Add `become: true` to all these tasks so they run with elevated privileges. Without this, the commands would hang and eventually time out after a few minutes.

### Ensure the latest version of requirements is installed

```yml
- name: Install required system packages
  ansible.builtin.apt:
    name:
      - ca-certificates
      - curl
    state: latest
    update_cache: true
  become: true
```

### Add the docker gpg key

```yml
- name: Add Docker GPG apt Key
  ansible.builtin.apt_key:
    url: https://download.docker.com/linux/ubuntu/gpg
    state: present
  become: true
```

### Add the Docker repository to apt sources

**Exercise:** Using the [Ansible `ansible.builtin.apt_repository` documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_repository_module.html), figure out how to write a task that adds the Docker repository (from the manual Docker install instructions) to your system. Don't forget to include `become: true`.

**Hint**: You can use `{{ ansible_distribution_release }}` to dynamically get your Ubuntu version name and use it as a variable.

### Update and install docker-ce

**Exercise:** Using the [Ansible `ansible.builtin.apt` documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html), figure out how to write a task that installs the latest `docker-ce` package.

### Create and add user to docker group

```yml
- name: Create "docker" group
  ansible.builtin.group:
    name: "docker"
    state: present
  become: true

- name: Add remote user to "docker" group
  ansible.builtin.user:
    name: "{{ username }}"
    group: "docker"
    append: yes
  become: true
```

**Note**: While only adding a single username to the docker group works, a more robust role would be designed to accept a list of usernames and loop through them. 

## Creating the Playbook

Now that our role is created with the installation logic, we need to create a playbook to execute it.

Create a new file `docker.yml` in your main project folder:

`docker.yml`

```yaml
---
- hosts: all
  vars:
    username: pluser
  roles:
    - docker
```

## Install Docker-ce using Ansible

Run the following command to install `docker-ce` on both hosts. 

```bash
ansible-playbook docker.yml
```

## Save to git

Time to save our progress!

```bash
git add .
git commit -m "Ansible install docker"
git push
```
