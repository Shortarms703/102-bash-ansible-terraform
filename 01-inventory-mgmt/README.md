# Inventory Management

## Inventory basics

The default location of the inventory file is under `/etc/ansible/hosts`. It is a standard practice to organize your folder structure by project and include the relevant inventory within the folder, and set that directory within the `ansible.cfg` file for the project. 

Below is an example of the project structure, which we will end up creating. 

```text
├── \
├── ansible.cfg
├── files
├── inventory
│   └── hosts.yml
├── roles
└── templates
```

Create the `hosts.yml` file. 

```bash
mkdir inventory
vi inventory/hosts.yml
```

`inventory/hosts.yml`

```yaml
nginx:
  hosts:
    webserver
ubuntu:
  hosts:
    webserver:
      ansible_host: {webserver IP}
    pki-server:
      ansible_host: {pki-server IP}
  vars:
    ansible_user: ansibleuser
```

Note that even though the `webserver` host is listed under both the ubuntu and nginx groups, it only needs to have it's IP address specified once. 

Any variable starting with `ansible_` is a built-in ansible variable. 

Create the `ansible.cfg` file. 

```bash
vi ansible.cfg
```

The `ansible.cfg` file is used to define configurations for ansible, including pointing to relevant file locations. The `inventory` key point to the directory (or specific file) where ansible should look for inventory files. 

Edit the `ansible.cfg` file to match the following. 

```ini
[defaults]
inventory = inventory
host_key_checking = False
```

The `host_key_checking = False` setting is not recommended for production environments, but is useful for development and testing purposes. This means it won't ask you to trust the host keys when connecting for the first time, like what happens when you SSH into a host for the first time.

We can test the inventory by pinging the hosts. 

```bash
ansible all -m ping
```

- `all`: Selects all hosts in the inventory
- `-m ping`: Uses the ping module to ping the hosts. Note that this does not run the "ping" command on the hosts, it is only used to determine if ansible can SSH to the hosts. 

The output will confirm that ansible was able to successfully SSH into the hosts and that the inventory is properly configured. 

Note that ansible uses the `ansible_user` variable to determine which user to use to SSH into the hosts (in this case `ansibleuser`). 

## Additional Information

You can also specify a particular inventory directory, or directories, using the `-i` flag when running your playbook:

`ansible-playbook get_logs.yml -i staging -i production`

You can create your inventory file in one of many formats, depending on the inventory plugins you have. The most common formats are INI and YAML. A basic INI `/etc/ansible/hosts` might look like this:

```ini
mail.domain.com

[webservers]
foo.domain.com
bar.domain.com

[dbservers]
one.domain.com
two.domain.com
three.domain.com
```

The INI format is not as easy to read or write as YAML, and it is not as flexible. We will use YAML for all of the following examples. 

Here's that same basic inventory file in YAML format.

```yaml
---
ungrouped:
  hosts:
    mail.domain.com:
webservers:
  hosts:
    foo.domain.com:
    bar.domain.com:
dbservers:
  hosts:
    one.domain.com:
    two.domain.com:
    three.domain.com:
```

Note that the `---` line is optional, and is used to separate several documents in a single YAML file. For this inventory, we only have one document, but it is often still included at the top of a YAML file.

## Groups

Hosts are organized into groups in the inventory file. Groups are used to organize hosts and to apply variables to hosts. Once organized into groups, we can then target specific groups of hosts when running different tasks. 

### Default Groups

Even if you do not define any groups in your inventory file, Ansible creates two default groups: `all` and `ungrouped`. The `all` group contains every host. The `ungrouped` group contains all hosts that don’t have another group aside from `all`. Every host will always belong to at least 2 groups (`all` and `ungrouped` or `all` and some other group).

### Hosts in Multiple Groups

You can put each host in more than one group. For example, a production web server in a data center in Toronto might be included in groups called `[prod]`, `[toronto]` and `[webservers]`

Extending the previous YAML inventory to include what, when, and where would look like this:

```yaml
ungrouped:
  hosts:
    mail.domain.com:
webservers:
  hosts:
    foo.domain.com:
    bar.domain.com:
dbservers:
  hosts:
    one.domain.com:
    two.domain.com:
    three.domain.com:
east:
  hosts:
    foo.domain.com:
    one.domain.com:
    two.domain.com:
west:
  hosts:
    bar.domain.com:
    three.domain.com:
prod:
  hosts:
    foo.domain.com:
    one.domain.com:
    two.domain.com:
test:
  hosts:
    bar.domain.com:
    three.domain.com:
```

In the example above, you can see that `one.domain.com` exists in the `dbservers`, `east`, and `prod` groups.

For our `inventory/hosts.yml` file, the `webserver` host is defined under the `nginx` and `ubuntu` groups. This means that `webserver` is a member of both the `nginx` and `ubuntu` groups.

### Grouping Groups: Parent/Child Group Relationships

You can create parent/child relationships among groups. Parent groups are also known as nested groups or groups of groups. For example, if all your production hosts are already in groups such as `toronto_prod` and `calgary_prod`, you can create a `production` group that includes those smaller groups. This approach reduces maintenance because you can add or remove hosts from the parent group by editing the child groups. To create parent/child relationships for groups, use the `children:` entry.

Here is the same inventory as shown above, simplified with parent groups for the `prod` and `test` groups. The two inventory files give you the same results:

```yaml
ungrouped:
  hosts:
    mail.domain.com:
webservers:
  hosts:
    foo.domain.com:
    bar.domain.com:
dbservers:
  hosts:
    one.domain.com:
    two.domain.com:
    three.domain.com:
east:
  hosts:
    foo.domain.com:
    one.domain.com:
    two.domain.com:
west:
  hosts:
    bar.domain.com:
    three.domain.com:
prod:
  children:
    east:
test:
  children:
    west:
```

## Variables in Inventory

You can store variable values that relate to a specific host or group in inventory. To start with, you may add variables directly to the hosts and groups in your main inventory file.

We document adding variables in the main inventory file for simplicity. However, storing variables in separate host and group variable files is a more robust approach to describing your system policy. Setting variables in the main inventory file is only a shorthand.

You can easily assign a variable to a single host and then use it later in playbooks. You can do this directly in your inventory file:

```yaml
toronto:
  hosts:
    host1:
      http_port: 80
      maxRequestsPerChild: 808
    host2:
      http_port: 303
      maxRequestsPerChild: 909
```

If all hosts in a group share a variable value, you can apply that variable to an entire group at once.

```yaml
toronto:
  hosts:
    host1:
    host2:
  vars:
    ntp_server: ntp.toronto.domain.com
    proxy: proxy.toronto.domain.com
```

You can apply variables to parent groups (nested groups or groups of groups) as well as to child groups using `vars:`. A child group’s variables will have higher precedence (override) than a parent group’s variables.

```yaml
canada:
  children:
    ontario:
      children:
        toronto:
          hosts:
            host1:
            host2:
        ottawa:
          hosts:
            host2:
            host3:
      vars:
        some_server: foo.ontario.example.com
        halon_system_timeout: 30
        self_destruct_countdown: 60
        escape_pods: 2
    quebec:
    alberta:
    bc:
```

### Save to git

Time to save our progress!

```bash
git add .
git commit -m "Inventory management"
git push

```
