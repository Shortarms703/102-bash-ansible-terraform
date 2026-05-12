# Ansible Automation of Linux Basics

As an exercise and for reusability go back to the [101-linux-certificates-git](https://github.com/PureLogicIT/101-linux-certificates-git) git repo and recreate everything we did there but in Ansible.

High Level Tasks:
* Rename host to use the ansible name as their hostname
* Patch the VMs
* Create the `courseadmin` group
* Give the group sudoers access to `apt-get`
* Create two new users (user1, user2)
* Add them to courseadmin group
* Add the CA certificate to the server
* Install NGINX on webserver
* Install ScribbleRS on webserver

Once done, don't forget to commit your changes to git
