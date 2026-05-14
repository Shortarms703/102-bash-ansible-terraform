# Ansible Vault

## What is an Ansible Vault?

Ansible Vault encrypts variables and files, allowing you to safeguard sensitive data like passwords or keys, instead of exposing them as plain text in playbooks or roles.

## Why use an Ansible Vault?

By using Ansible Vault, you ensure that sensitive information is safeguarded while maintaining the flexibility and efficiency of automation.

**Use Cases:**

- **Database Credentials:** Securely store database passwords or connection strings.
- **API Keys and Tokens:** Encrypt API keys for third-party services.
- **SSL/TLS Certificates:** Safeguard private keys and certificates required for secure communication.
- **SSH Keys:** Store SSH private keys used for accessing servers.

## How to create an Ansible Vault

Before we create the vault we'll create a `group_vars` folder. Ansible automatically loads the files in this directory as variable files. The variables are assigned to hosts by group name.

**Note:** Ansible's variable precedence (the order in which variables load) can be confusing. The idea is that you can put variables in many different places, but `group_vars/` is one of the most likely and organized places to store them.

```bash
mkdir -p group_vars/ubuntu
```

To create a new Ansible Vault file, use the `ansible-vault` command with the `create` option:

```bash
ansible-vault create group_vars/ubuntu/vault.yml
```

- `create` creates a brand new vault file. You will be prompted to set a password to encrypt it. This password will be required to access or edit the vault file.
- Once the vault file is created, your default text editor (e.g., nano, vi) will open so that you can add details in YAML format.

```yaml
vault_ansibleuser_password: <PASSWORD>
```

- Add the above text, then save and close the file. 

View the contents of the vault file. 

```bash
cat group_vars/ubuntu/vault.yml
```

From the output, we can see the file is encrypted with AES256, as we expect. 

## How to decrypt an Ansible Vault

To decrypt an Ansible Vault file, use the `ansible-vault` command with the `decrypt` option. We'll do this just to make sure we can successfully decrypt it:

```bash
ansible-vault decrypt group_vars/ubuntu/vault.yml
```

View the `vault.yml` file to see the contents of the file. 

```bash
cat group_vars/ubuntu/vault.yml
```

**Note:** The Ansible Vault file is converted back to plain text with the above decrypt command. **Use with caution**. After decrypting, you can modify the file directly, and then re-encrypt it. Ensure that the file is re-encrypted once you have finished reviewing/modifying it.

Re-encrypt the file with the following command:

```bash
ansible-vault encrypt group_vars/ubuntu/vault.yml
```

## How to edit an Ansible Vault

To edit an Ansible Vault file, use the `ansible-vault` command with the `edit` option:

```bash
ansible-vault edit group_vars/ubuntu/vault.yml
```

This allows you to edit the file without having to decrypt and re-encrypt it. Using `edit` is nice because you don't need to remember to re-encrypt the file afterwards. If we decrypt a file and push it to git, we might forget to re-encrypt it and leak secrets, so always using `edit` is good practice.

## How to use values in an Ansible Vault

Create a normal non-encrypted file that will call the encrypted vault variable. This gives us a single file to find all the variables later. When you look at the password here, you can clearly see it comes from the vault (because of the `vault_` prefix), and it can be stored in the same place as all your other passwords and variables.

While creating this file we'll also set the `ansible_user` variable, which will set the username ansible uses. 

```bash
vi group_vars/ubuntu/vars
```

`group_vars/ubuntu/vars`

```ini
ansible_user: ansibleuser
ansibleuser_password: "{{vault_ansibleuser_password}}"
```

**Note:** It is sometimes confusing, since you may think `ansibleuser_password` will be automatically used by Ansible just because it starts with "ansible-". However, `ansibleuser_password` and `vault_ansibleuser_password` are just custom variable names we made up—they aren't special. Built-in variables like `ansible_become_pass` and `ansible_user`, on the other hand, *are* special and recognized by Ansible automatically.

Modify the `inventory/hosts.yml` file to use the new vault password

```yaml
nginx:
  hosts:
    webserver
ubuntu:
  hosts:
    webserver:
      ansible_host: webserver
      ansible_become_pass: "{{ ansibleuser_password }}"
    pki-server:
      ansible_host: pki-server
      ansible_become_pass: "{{ ansibleuser_password }}"
  vars:
    ansible_user: ansibleuser
```

Now we can test it. 

```bash
ansible -m debug -a 'var=hostvars[inventory_hostname]' --ask-vault-password ubuntu
```

You should see the newly set variables in the output, including our vault values

```json
webserver | SUCCESS => {
    "hostvars[inventory_hostname]": {
        "ansible_become_pass": "pluser!",
        ...
        "ansible_host": "webserver",
        ...
        "ansible_user": "ansibleuser",
        ...
        "ansibleuser_password": "pluser!",
        ...
        "vault_ansibleuser_password": "pluser!"
    }
```

- `ansible_become_pass`: A reserved keyword in Ansible that is used to store the password for privilege escalation. 
- `ansibleuser_pass`: This however, is just the variable we created, and is not a reserved keyword, even though it starts with "ansible". 

Writing the vault password each time can get tedious. Create a vault password file and enter your vault password.

```bash
vi .vault_password
```

Then, edit `ansible.cfg` and set the file

`ansible.cfg`

```ini
[defaults]
inventory = inventory
host_key_checking = False
vault_password_file = .vault_password
```

Run the same command as before. 

```bash
ansible -m debug -a 'var=hostvars[inventory_hostname]' --ask-vault-password ubuntu
```

Notice that we are still prompted to enter our vault password. This is because we are explicitly telling Ansible to ask for it by using `--ask-vault-password`. Remove this option to stop being prompted for the vault password.

```bash
ansible -m debug -a 'var=hostvars[inventory_hostname]' ubuntu
```

## Save progress to git

Add `.vault_password` to your `.gitignore` so that you never accidentally commit it. This is good practice for any sensitive file.

```bash
echo ".vault_password" >> .gitignore
git add .
git commit -m "Ansible vault section"
git push
```
