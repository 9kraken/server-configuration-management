# Server Configuration Management

Automated server configuration and deployment for **raskumarbig.ru** using Ansible roles.

## Overview

The goal is to write an Ansible playbook (`setup.yml') and create the following roles:

* `base` — basic server setup (installs utilities, updates the server, installs fail2ban, etc.)

* `nginx` — installs and configures nginx

* `app` — uploads the given tarball of a static HTML website to the server and unzips it.  
(implemented via `rsync` synchronization instead of a tarball for better performance and differential updates)

* `ssh` - adds the given public key to the server

<https://roadmap.sh/projects/configuration-management>

## Project Structure

```text
.
|___ansible.cfg
|___inventory.yml
|___LICENSE
|___README.md
|___setup.yml
|___group_vars
| |___all.yml
|___roles
| |___app
| | |___tasks
| |   |___main.yml
| |
| |___base
| | |___tasks
| |   |___main.yml
| |
| |___nginx
| | |___handlers
| | | |___main.yml
| | |
| | |___tasks
| |   |___main.yml
| |
| |___ssh
|   |___tasks
|     |___main.yml
```

**Note:** Group variables are stored in `group_vars/all.yml` and loaded automatically by Ansible.

## Prerequisites

1. Linux server running.
2. User with **sudo** privileges on it.
3. SSH key with **600** permissions and **no passphrase***

* The sudo password is provided only once via `-K` flag.  
However, `synchronize` establishes parallel SSH session and requires the private key path  
to be explicitly defined to proceed without a passphrase prompt.

## Requirements

1. Ansible

## Usage

### 1. Configure Variables

Create `group_vars/all.yml` with the following parameters:

```yaml
local_site_path: "/path/to/website/"        # Local path to your website files
remote_site_path: "/var/www/website/"       # Remote path where site will be deployed
site_domain: "example.com"                  # Your domain name
site_folder_name: "website"                 # Folder name for site configuration
site_port: "8443"                           # Port for the website
ssh_key_path: "~/.ssh/your_key"             # Path to your private SSH key (no passphrase)
ssh_public_key_path: "~/.ssh/your_key.pub"  # Path to your public SSH key
ssh_user: "deploy_user"                     # SSH user on the server
target_host: "your.server.ip"               # Server IP address or hostname
```

### 2. Run Ansible Playbook

Variables from `group_vars/all.yml` are loaded automatically.

To run the full setup:

```bash
ansible-playbook -i inventory.yml setup.yml -K
```

To deploy only specific roles:

```bash
ansible-playbook -i inventory.yml setup.yml -K --tags base
ansible-playbook -i inventory.yml setup.yml -K --tags nginx
ansible-playbook -i inventory.yml setup.yml -K --tags app
ansible-playbook -i inventory.yml setup.yml -K --tags ssh
```
