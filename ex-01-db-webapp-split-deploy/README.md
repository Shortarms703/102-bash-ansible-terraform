# Database and Web Application Deployment

student A and student B

## Git repo and secrets

Create a git repo, share with both members

create a hosts file that defines two groups: db_server and web_server, assigning respective VM IPs to each. make the IP of db_server be a VM of student A, and web_server be a vm of student B

`inventory/hosts.yml`

```yaml
all:
  children:
    # Student A's
    db_server:
      hosts:
        db_vm_01:
          ansible_host: pki-server

    # Student B's
    web_server:
      hosts:
        web_vm_01:
          ansible_host: webserver

  vars:
    ansible_user: ansibleuser
    ansible_become_pass: "{{ vault_sudo_password }}"
```

`ansible.cfg`

```ini
[defaults]
inventory = inventory
host_key_checking = False
vault_password_file = ./.vault_password
```

define a vars file, with `wp_db_name`, `wp_db_user`, and `db_public_ip` variables. 

`group_vars/all/vars.yml`

```yaml
wp_db_name: wordpressdb
wp_db_user: wp_admin
db_public_ip: {DB_SERVER public IP}
```

use ansible-vault to store the database password

`ansible-vault create group_vars/all/vault.yml`

```yaml
vault_wp_db_password: { password }
vault_sudo_password: { password }
```

Create `.vault_password` file. Have it only contain the vault password. 

Add the .vault_password file to .gitignore

```bash
echo ".vault_password" >> .gitignore
```

Give ansibleuser sudo on both `webserver` and `pki-server`. 

```bash
sudo usermod -aG sudo ansibleuser
```

Commit everything to Git. A and B should both have the git repo at the same point. 

## Student A - Database Layer

write a playbook `deploy_db.yml` that targets the `db_server` group. 

`deploy_db.yml`

```yaml
- name: Deploy MariaDB for WordPress
  hosts: db_server
  become: yes

  tasks:
    - name: Install MariaDB and required Python library
      ansible.builtin.apt:
        name:
          - mariadb-server
          - python3-pymysql # Required by Ansible's mysql modules
        state: present
        update_cache: yes

    - name: Configure MariaDB to bind to all interfaces
      ansible.builtin.lineinfile:
        path: /etc/mysql/mariadb.conf.d/50-server.cnf
        regexp: '^bind-address\s*='
        line: 'bind-address = 0.0.0.0'
        backup: yes
      notify: Restart MariaDB

    - name: Force MariaDB to restart before creating the database
      meta: flush_handlers

    - name: Ensure MariaDB service is started and enabled
      ansible.builtin.systemd:
        name: mariadb
        state: started
        enabled: yes

    - name: Create the WordPress database
      community.mysql.mysql_db:
        name: "{{ wp_db_name }}"
        state: present
        login_unix_socket: /var/run/mysqld/mysqld.sock

    - name: Create the database user and grant permissions
      community.mysql.mysql_user:
        name: "{{ wp_db_user }}"
        password: "{{ vault_wp_db_password }}"
        priv: "{{ wp_db_name }}.*:ALL"
        host: "%" # Allows connection from any IP. For strict security, replace '%' with Student B's IP.
        state: present
        login_unix_socket: /var/run/mysqld/mysqld.sock

  handlers:
    - name: Restart MariaDB
      ansible.builtin.systemd:
        name: mariadb
        state: restarted
```

## Student B - Web Layer

`deploy_web.yml`

```yaml
- name: Deploy WordPress Web Server
  hosts: web_server
  become: yes

  tasks:
    - name: Install Nginx, PHP, and required extensions
      ansible.builtin.apt:
        name:
          - nginx
          - php-fpm
          - php-mysql
          - tar
        state: present
        update_cache: yes

    - name: Download and extract WordPress
      ansible.builtin.unarchive:
        src: https://wordpress.org/latest.tar.gz
        dest: /var/www/html
        remote_src: yes
        creates: /var/www/html/wordpress/wp-settings.php # Prevents re-downloading if it already exists

    - name: Set ownership of WordPress directory
      ansible.builtin.file:
        path: /var/www/html/wordpress
        state: directory
        owner: www-data
        group: www-data
        recurse: yes

    - name: Deploy wp-config.php from template
      ansible.builtin.template:
        src: templates/wp-config.php.j2
        dest: /var/www/html/wordpress/wp-config.php
        owner: www-data
        group: www-data
        mode: '0644'

    - name: Deploy Nginx configuration for WordPress
      ansible.builtin.template:
        src: templates/default.j2
        dest: /etc/nginx/sites-available/default
        owner: root
        group: root
        mode: '0644'
      notify: Restart Nginx

    - name: Ensure Nginx is started and enabled
      ansible.builtin.systemd:
        name: nginx
        state: started
        enabled: yes

  handlers:
    - name: Restart Nginx
      ansible.builtin.systemd:
        name: nginx
        state: restarted
```


`templates/wp-config.php.j2`

```php
<?php
// ** Database settings ** //
define( 'DB_NAME', '{{ wp_db_name }}' );
define( 'DB_USER', '{{ wp_db_user }}' );
define( 'DB_PASSWORD', '{{ vault_wp_db_password }}' );
define( 'DB_HOST', '{{ db_public_ip }}' );

define( 'DB_CHARSET', 'utf8' );
define( 'DB_COLLATE', '' );

$table_prefix = 'wp_';

define( 'WP_DEBUG', false );

/** Absolute path to the WordPress directory. */
if ( ! defined( 'ABSPATH' ) ) {
        define( 'ABSPATH', __DIR__ . '/' );
}

/** Sets up WordPress vars and included files. */
require_once ABSPATH . 'wp-settings.php';
```

`templates/default.j2`

```nginx
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    # Point the root directly to the wordpress folder to make the URL cleaner!
    root /var/www/html/wordpress;

    # Add index.php to the front of the list
    index index.php index.html index.htm index.nginx-debian.html;

    server_name _;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    # Pass PHP scripts to FastCGI server
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;

        # NOTE: If you are using Ubuntu 24.04, change 8.1 to 8.3!
        fastcgi_pass unix:/run/php/php8.3-fpm.sock;
    }
}
```

## Deployment and Verification

commit A and B's code

run the playbooks, maybe both students individually on their VM groups, so they have to change the variable and give sudo perms to ansibleuser on both
