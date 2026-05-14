# Ansible Patching

This Ansible playbook is targeting a Ubuntu host and performs the following sequence:

1. First, checking for pending updates.
2. Then actually look for and install updates (update, upgrade, autoremove).
3. Checks if there's a reboot required, since sometimes updates have a reboot required after they install.
4. Then run the reboot if necessary.

```yaml
---
- hosts: all
  serial: 2 # Serial is used to control how many hosts are managed at a time, in this case 2 at a time. Without specifying this, the default will run against all hosts in parallel.
  tasks:
    - name: Checking for pending updates
      command: /usr/lib/update-notifier/apt-check --package-names # This command checks for packages that have available updates and then lists the names, and registers those package names to the variable "packages"
      register: packages
      changed_when: packages.stderr != "" # This will only result in a "changed" status reported by ansible when the output from stderr is not blank, so there will be no changes reported if there are no new package updates available

    - name: update, upgrade, autoremove
      become: yes # Become root user for the task
      apt: 
        update_cache: yes # This is the equivalent of apt-get update, which gets info about latest package versions and dependencies
        upgrade: full # This is the equivalent of running apt-get upgrade. The other options here are no, dist, and safe.
        autoremove: yes # This removes unused dependency packages
      when: packages.stderr != "" # This line sets the condition on when to run this task, in this case it only runs if there were no errors reported in the "Checking for pending updates" task

    - name: Check if a reboot is required
      stat:
        path: /var/run/reboot-required # This file only exists in Ubuntu when a reboot is required
      register: reboot_required
      changed_when: reboot_required.stat.exists

    - name: Reboot system if required
      become: yes
      reboot:
        reboot_timeout: 900 # This is the maximum amount of seconds to wait for a reboot to happen and have the system respond to a test command
      when: reboot_required.stat.exists == true # This task will only happen based on the registered variable "reboot_required" result from the last task
      register: restart_status
```

## Cleaning up the file with Vim Macros

The playbook above has a lot of inline comments (starting with `#`), which can make it a bit hard to read. Let's use Vim macros to clean it up.

Copy the playbook above into your file using `vi patching.yml`.

Then, run the following VIM macros to remove the comments. Enter each of the following right after opening the `patching.yml` file:

1. `q` - enter macro recording mode
2. `a` - name the macro "a"
3. `/^#` - search for a comment character at the beginning of a line
4. `<Press Enter>` - execute the search
5. `d$` - delete from the cursor to the end of the line
6. `q` - finish recording the macro

Now that the macro `a` is recorded, you can execute it multiple times to clean up the rest of the file.

Type `@a` to run the macro once, or `20@a` to run the macro 20 times. It will jump to each `#` and delete the comment. 

## Referencing Ansible Documentation

As you work with these modules, it's highly recommended to look at the official Ansible documentation for all of them.

- `stat` module: Go look at the Ansible docs for `ansible.builtin.stat`. Skip down to the Examples section and find "Get stats of the FS object" under the `Examples` section. You'll see how we use `stat`. Then, look under the `Return Values` section for `stat.exists`.
- `apt` module: Look in the `Examples` section of the `ansible.builtin.apt` module. You'll find standard update/upgrade tasks in the examples, similar to the one in our patching task. 
- `state` parameter: For many Ansible modules, the `state` parameter is a pretty important one, and `state: present` will almost always be there as a default, under the parameters section of the documentation.

## Save to git

Time to save our progress!

```bash
git add .
git commit -m "Ansible Patching"
git push
```
 