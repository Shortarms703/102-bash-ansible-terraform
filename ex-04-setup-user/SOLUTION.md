# Solution: Setting up a New User with SSH Access

Here is the complete `setup_user.yml` playbook containing the tasks required to create the user, copy the SSH key, and configure passwordless sudo.

`setup_user.yml`

```yaml
---
- name: Provision a new deploy user
  hosts: all
  become: yes
  vars:
    new_user: "deployuser"
    ssh_pub_key: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"

  tasks:
    - name: Ensure the user exists with a home directory and bash shell
      ansible.builtin.user:
        name: "{{ new_user }}"
        shell: /bin/bash
        create_home: yes
        state: present

    - name: Set authorized SSH key for the new user
      ansible.posix.authorized_key:
        user: "{{ new_user }}"
        state: present
        key: "{{ ssh_pub_key }}"

    - name: Grant passwordless sudo to the new user
      ansible.builtin.copy:
        content: "{{ new_user }} ALL=(ALL) NOPASSWD: ALL"
        dest: "/etc/sudoers.d/{{ new_user }}"
        owner: root
        group: root
        mode: '0440'
        # Crucial: Always validate sudoers files before applying them!
        validate: 'visudo -cf %s'
```
