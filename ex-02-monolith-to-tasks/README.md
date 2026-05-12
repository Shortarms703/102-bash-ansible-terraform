# Transform Monolith Ansible Playbook to set of Tasks

Switch roles, so both people will have seen web and db parts

take the two big playbook files, split into tasks

`ansible-galaxy init roles/wordpress_web`

^ builds scaffolding

deconstruct both, same for db

now playbook can just look like this:

`site-playbook.yml`

```yaml
---
# Play 1: Setup the Database
- name: Deploy the Database Tier
  hosts: db_server
  become: yes
  roles:
    - wordpress_db

# Play 2: Setup the Web Application
- name: Deploy the Web Tier
  hosts: web_server
  become: yes
  roles:
    - wordpress_web
```

talk about how this makes it better overall