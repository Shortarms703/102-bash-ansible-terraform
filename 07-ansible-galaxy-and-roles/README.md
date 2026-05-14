# Understanding Roles and Ansible Galaxy

## What are Roles in Ansible?

Ansible roles are a way to organize and modularize your configuration and automation tasks. They allow you to break down your playbooks into reusable and structured components. Each role encapsulates a specific functionality, making it easier to manage and reuse tasks, variables, files, and templates across multiple playbooks or projects.

Roles are stored in directories with a standardized structure, which Ansible automatically recognizes and uses during execution.

## How are Roles used in Ansible?

Roles are typically used in playbooks. Instead of defining all tasks, variables, and files directly in the playbook, you can include roles to simplify your playbooks.

Here’s how you would use a role in an Ansible playbook:

```yaml
- hosts: webservers
  roles:
    - apache
```

In this example, the `apache` role will execute all tasks, use any variables, and apply configurations files defined in its directory.

### Role Directory Structure

A typical Ansible role has the following structure:

```python
roles/
  apache/
    tasks/
      main.yml       # Main task file
    handlers/
      main.yml       # Handlers for service notifications
    templates/
      apache.conf.j2 # Jinja2 templates
    files/
      index.html     # Static files
    vars/
      main.yml       # Role-specific variables
    defaults/
      main.yml       # Default variables
    meta/
      main.yml       # Role metadata
```

- **`tasks/`**: The most important folder. This contains the actual tasks that run.
- **`handlers/`**: Used to call tasks when something else finishes (e.g., restarting a service or cleaning up after a crash). We won't cover these in depth.
- **`templates/`**: When you reference a template in a task, Ansible looks here first.
- **`files/`**: When you reference a file in a task, Ansible looks here first, before checking other spots.
- **`vars/`**: Variables defined here are strict and **not** intended to be overridden.
- **`defaults/`**: Default variables that **are** intended to be overridden.
- **`meta/`**: Contains metadata and comments for documentation. We won't cover these in depth.

### Creating a Role

Convert our previous patching playbook into an organized role.

First, use `ansible-galaxy init` to automatically generate the standard role directory structure for us:

```bash
mkdir roles
cd roles
ansible-galaxy init patching
cd ..
```

To see the structure it created, run:

```bash
tree -d roles/patching
```

Next, open the previous `patching.yml` file from the patching exercise, copy the tasks, and paste them into `roles/patching/tasks/main.yml`.

When you paste them, they will likely have 4 spaces of indentation from being nested in the old playbook. Remove those spaces so the `- name:` list aligns to the left margin. You can quickly remove the leading spaces using this Vim substitution command:

```vim
:%s/^    //g
```

- As you enter the `spaces`, you will see them highlighting the text that will be selected. You can use this as a judge for how many spaces to enter.

Now that your role is fully created, let's make a brand new `patching.yml` playbook at the root of your project that is much cleaner.

`patching.yml`

```yaml
---
- hosts: all
  serial: 2
  roles:
    - patching
```

- `serial: 2`: This tells Ansible to process no more than 2 hosts in parallel at a time, which is useful for safe rolling updates.

Run this new playbook:

```bash
ansible-playbook patching.yml
```

It will work exactly like before, but now the logic is encapsulated entirely within a reusable role! We will be using this role later in the course.

## Ansible Galaxy

**Ansible Galaxy** is a platform for sharing, discovering, and downloading Ansible roles and collections. It simplifies the process of finding reusable roles created by the Ansible community or official contributors. Instead of writing every role from scratch, you can search for pre-written roles on Ansible Galaxy that meet your needs, saving time and effort.

*Note: While Ansible Galaxy points to the public community repository by default, organizations can configure it to point to their own local or private Galaxy server.*

### Using Ansible Galaxy

1. **Search for Roles:** You can search for roles on the [Ansible Galaxy](https://galaxy.ansible.com/) website.

2. **Install a Role:** Use the `ansible-galaxy` command to download and install a role. For example:

    ```bash
    ansible-galaxy install geerlingguy.apache
    ```

3. **Use the Installed Role:** Include the role in your playbook just like any other role. 

#### Example Workflow

```bash
# Install a role from Galaxy
ansible-galaxy install geerlingguy.apache

# Include the role in your playbook
- hosts: all
  roles:
    - geerlingguy.apache
``` 

## Save to git

Time to save our progress!

```bash
git add .
git commit -m "Ansible galaxies and roles"
git push
```
