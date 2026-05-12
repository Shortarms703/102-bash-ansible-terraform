# Installing Ansible

The recommended way to install Ansible is with `pip` or `pipx`

1. Install pipx

   ```bash
   sudo apt update
   sudo apt install pipx
   pipx ensurepath
   ```

2. Install ansible

   ```bash
   pipx install --include-deps ansible
   ```

3. Logout and log back in to update the paths

## Create Ansible repo in Git

Lets save all our work in git to ensure no loss of work in the case the VM goes down.

1. Create a repository

   Go to Gitea and create a repository called AnsibleCourse_{username}

2. Create local repository and connect to remote

```bash
   mkdir ansible
   cd ansible
   touch README.md
   git init
   git checkout -b main
   git add README.md
   git commit -m "first commit"
   git remote add origin <remote_repository_URL>
   git push -u origin main
```
