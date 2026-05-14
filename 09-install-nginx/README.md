# Install NGINX and upload new index.html file

We will be performing the same task we did in `101 - Linux Basics`, but using Ansible to automate the tasks instead. 

Just like with Docker, we are going to build this as a reusable **role** rather than a flat playbook.

First, initialize the new role:

```bash
cd roles
ansible-galaxy init nginx
cd ..
```

All the tasks below should be placed into your `roles/nginx/tasks/main.yml` file.

## Automated with Ansible

Write the tasks for this role by reading the Ansible documentation. 

**Note**: Remember to add `become: true` to the tasks since installing packages and managing services requires elevated privileges.

### Install nginx

Using the [Ansible `ansible.builtin.apt` documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html), write a task to install the latest `nginx` package and update the apt cache.

### Start the nginx service

Using the [Ansible `ansible.builtin.systemd_service` documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/systemd_service_module.html), write a task to ensure the `nginx` service is both `started` and `enabled` to run on boot.

### Copy the website files

Using the [Ansible `ansible.builtin.copy` documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html), write a task to copy an `index.html` file from your Ansible client to the target machine's NGINX directory (`/var/www/html/index.html`). Make sure the owner and group are set to `root`.

# Need tristan's website here, so that it can be copied, however we want to do that

*(have the students set up the website files again exactly like they did before in the Linux Basics exercise!)*

## Creating the Playbook

Now that our role is created with the installation logic, we need to create a playbook to execute it.

Create a new file `nginx.yml` in your main project folder:

`nginx.yml`

```yaml
---
- hosts: nginx
  become: true
  roles:
    - nginx
```

## Install Nginx using Ansible

Run the following command to execute your new playbook!

```bash
ansible-playbook nginx.yml
```

## Save to git

Time to save our progress!

```bash
git add .
git commit -m "install nginx"
git push
```
