# Ansible

## What is Ansible?

**Ansible** is an open-source automation and configuration management tool.

It is used to:

* Automate server configuration
* Install and configure software
* Deploy applications
* Manage multiple servers
* Automate repetitive tasks
* Manage cloud infrastructure
* Perform continuous deployment

Ansible uses a **Control Node** to communicate with **Managed Nodes** using SSH.

---

# Ansible Architecture

```text
                    Ansible Control Node
                  (Ansible Installed Here)
                           |
                           | SSH
             --------------+--------------
             |                            |
             ↓                            ↓
      Managed Node 1              Managed Node 2
       (Web Server)                 (DB Server)
```

### Control Node

The machine where Ansible is installed.

We will run:

* Ansible commands
* Ad-hoc commands
* Playbooks
* Roles
* Vault
* Configuration tasks

### Managed Node

The server that Ansible manages.

Ansible does **not** need to be installed on the managed node.

---

# Lab Setup

For this lab, create **2 Ubuntu EC2 instances**.

| Instance | Purpose      | Ansible      |
| -------- | ------------ | ------------ |
| EC2-1    | Control Node | Installed    |
| EC2-2    | Managed Node | Not required |

```text
EC2-1
Control Node
Ansible Installed
       |
       | SSH
       ↓
EC2-2
Managed Node
Ansible Not Required
```

---

# 1. Ansible Installation

## Install Ansible on Control Node

Run these commands on **EC2-1**.

### Update packages

```bash
sudo apt update
```

### Install Ansible

```bash
sudo apt install ansible -y
```

### Verify Ansible

```bash
ansible --version
```

### Check Python

```bash
python3 --version
```

---

# 2. Configure SSH

Ansible uses SSH to connect to the managed node.

From the **Control Node**, test SSH connection:

```bash
ssh ubuntu@<MANAGED_NODE_PRIVATE_IP>
```

Example:

```bash
ssh ubuntu@172.31.20.50
```

If SSH works, exit the managed node:

```bash
exit
```

---

# 3. Inventory

An **Inventory** file contains information about the servers that Ansible manages.

Create a directory:

```bash
mkdir ansible-lab
```

Go inside:

```bash
cd ansible-lab
```

Create inventory file:

```bash
nano inventory
```

Add:

```ini
[webservers]
172.31.20.50 ansible_user=ubuntu
```

Replace:

```text
172.31.20.50
```

with your managed node private IP.

---

# Test Inventory

Check the inventory:

```bash
ansible-inventory -i inventory --list
```

List hosts:

```bash
ansible -i inventory all --list-hosts
```

---

# 4. Ansible Ad-Hoc Commands

Ad-hoc commands are used to perform quick tasks without creating a playbook.

## Test Connection

```bash
ansible -i inventory webservers -m ping
```

Expected result:

```text
SUCCESS
```

---

## Check Uptime

```bash
ansible -i inventory webservers -m command -a "uptime"
```

---

## Check Disk Usage

```bash
ansible -i inventory webservers -m command -a "df -h"
```

---

## Check Memory

```bash
ansible -i inventory webservers -m command -a "free -h"
```

---

## Check OS Information

```bash
ansible -i inventory webservers -m command -a "cat /etc/os-release"
```

---

# Install Nginx Using Ad-Hoc Command

```bash
ansible -i inventory webservers -b -m apt -a "name=nginx state=present"
```

Start Nginx:

```bash
ansible -i inventory webservers -b -m service -a "name=nginx state=started"
```

---

# 5. Ansible Playbooks

A **Playbook** is a YAML file containing multiple Ansible tasks.

Create:

```bash
nano nginx.yml
```

Add:

```yaml
---
- name: Install and configure Nginx
  hosts: webservers
  become: yes

  tasks:

    - name: Install Nginx
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Start Nginx
      service:
        name: nginx
        state: started
        enabled: yes
```

Run the playbook:

```bash
ansible-playbook -i inventory nginx.yml
```

---

# Verify Nginx

On the managed node:

```bash
systemctl status nginx
```

Check Nginx:

```bash
curl http://localhost
```

---

# 6. Ansible Modules

Modules are the building blocks of Ansible.

Common modules:

| Module     | Purpose                           |
| ---------- | --------------------------------- |
| apt        | Install packages on Debian/Ubuntu |
| yum        | Install packages on RHEL/CentOS   |
| dnf        | Package management                |
| service    | Manage services                   |
| systemd    | Manage systemd services           |
| copy       | Copy files                        |
| file       | Manage files/directories          |
| user       | Manage users                      |
| group      | Manage groups                     |
| command    | Execute commands                  |
| shell      | Execute shell commands            |
| debug      | Display information               |
| template   | Copy Jinja2 templates             |
| lineinfile | Modify lines in files             |

---

# Module Examples

## Create Directory

```bash
ansible -i inventory webservers -b -m file -a "path=/opt/devops state=directory"
```

## Create File

```bash
ansible -i inventory webservers -b -m file -a "path=/opt/devops/test.txt state=touch"
```

## Copy File

Create a local file:

```bash
echo "Hello from Ansible" > test.txt
```

Copy it to the managed node:

```bash
ansible -i inventory webservers -b -m copy -a "src=test.txt dest=/tmp/test.txt"
```

---

# 7. Variables

Variables are used to store values that can be reused.

Create:

```bash
nano variables.yml
```

Add:

```yaml
---
- name: Variables Example
  hosts: webservers
  become: yes

  vars:
    package_name: nginx
    service_name: nginx

  tasks:

    - name: Install package
      apt:
        name: "{{ package_name }}"
        state: present

    - name: Start service
      service:
        name: "{{ service_name }}"
        state: started
```

Run:

```bash
ansible-playbook -i inventory variables.yml
```

---

# 8. Ansible Facts

Facts are information automatically collected from managed nodes.

Run:

```bash
ansible -i inventory webservers -m setup
```

Some useful facts:

```text
ansible_hostname
ansible_distribution
ansible_os_family
ansible_default_ipv4.address
ansible_processor_vcpus
ansible_memtotal_mb
```

Example:

```bash
ansible -i inventory webservers -m setup -a "filter=ansible_hostname"
```

---

# 9. Conditionals

Conditionals allow Ansible to execute a task only when a condition is true.

Example:

```yaml
---
- name: Conditional Example
  hosts: webservers
  become: yes

  tasks:

    - name: Install Nginx on Ubuntu
      apt:
        name: nginx
        state: present
      when: ansible_distribution == "Ubuntu"
```

Run:

```bash
ansible-playbook -i inventory conditional.yml
```

Another example:

```yaml
when: ansible_os_family == "RedHat"
```

---

# 10. Loops

Loops are used when we want to perform the same task for multiple items.

Example:

```yaml
---
- name: Loop Example
  hosts: webservers
  become: yes

  tasks:

    - name: Install multiple packages
      apt:
        name: "{{ item }}"
        state: present
      loop:
        - nginx
        - git
        - curl
        - wget
```

Run:

```bash
ansible-playbook -i inventory loops.yml
```

---

# 11. Handlers

Handlers are tasks that run when they are notified by another task.

They are commonly used for restarting or reloading services after a configuration change.

Example:

```yaml
---
- name: Handler Example
  hosts: webservers
  become: yes

  tasks:

    - name: Update Nginx configuration
      copy:
        content: |
          server {
              listen 80;
              location / {
                  return 200 "Hello from Ansible";
              }
          }
        dest: /etc/nginx/sites-available/default
      notify: Restart Nginx

  handlers:

    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
```

Run:

```bash
ansible-playbook -i inventory handlers.yml
```

---

# 12. Templates - Jinja2

Templates allow us to create dynamic configuration files.

Ansible uses **Jinja2** for templating.

Create templates directory:

```bash
mkdir templates
```

Create template:

```bash
nano templates/index.html.j2
```

Add:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Ansible Server</title>
</head>
<body>

<h1>Hello from Ansible</h1>

<p>Hostname: {{ ansible_hostname }}</p>
<p>Operating System: {{ ansible_distribution }}</p>
<p>IP Address: {{ ansible_default_ipv4.address }}</p>

</body>
</html>
```

Create playbook:

```bash
nano template.yml
```

Add:

```yaml
---
- name: Template Example
  hosts: webservers
  become: yes

  tasks:

    - name: Install Nginx
      apt:
        name: nginx
        state: present

    - name: Copy HTML template
      template:
        src: templates/index.html.j2
        dest: /var/www/html/index.html

    - name: Start Nginx
      service:
        name: nginx
        state: started
```

Run:

```bash
ansible-playbook -i inventory template.yml
```

Now open the managed node's public IP in a browser.

---

# 13. Ansible Roles

Roles are used to organize large Ansible projects.

Create a role:

```bash
ansible-galaxy init nginx
```

Directory structure:

```text
nginx/
├── defaults/
│   └── main.yml
├── handlers/
│   └── main.yml
├── tasks/
│   └── main.yml
├── templates/
├── files/
├── vars/
│   └── main.yml
├── meta/
│   └── main.yml
└── README.md
```

Important directories:

| Directory | Purpose           |
| --------- | ----------------- |
| tasks     | Main tasks        |
| handlers  | Handlers          |
| templates | Jinja2 templates  |
| files     | Static files      |
| defaults  | Default variables |
| vars      | Variables         |
| meta      | Role metadata     |

---

# Role Example

Create:

```bash
ansible-galaxy init nginx
```

Edit:

```bash
nano nginx/tasks/main.yml
```

Add:

```yaml
---
- name: Install Nginx
  apt:
    name: nginx
    state: present

- name: Start Nginx
  service:
    name: nginx
    state: started
    enabled: yes
```

Create playbook:

```bash
nano role.yml
```

Add:

```yaml
---
- name: Nginx Role
  hosts: webservers
  become: yes

  roles:
    - nginx
```

Run:

```bash
ansible-playbook -i inventory role.yml
```

---

# 14. Ansible Vault

Ansible Vault is used to encrypt sensitive information such as:

* Passwords
* API keys
* Database credentials
* Secrets

Create encrypted file:

```bash
ansible-vault create secrets.yml
```

Example:

```yaml
db_username: admin
db_password: MySecretPassword
```

Save and exit.

---

# Edit Vault

```bash
ansible-vault edit secrets.yml
```

View Vault:

```bash
ansible-vault view secrets.yml
```

Change Vault password:

```bash
ansible-vault rekey secrets.yml
```

---

# Run Playbook With Vault

```bash
ansible-playbook -i inventory vault.yml --ask-vault-pass
```

**Important:** Never upload passwords, private keys, or unencrypted secrets to GitHub.

---

# 15. Terraform + Ansible

Terraform and Ansible can be used together.

### Terraform

Terraform is mainly used to create infrastructure.

```text
Terraform
   |
   ↓
VPC
EC2
Security Groups
Load Balancer
```

### Ansible

Ansible is mainly used to configure the infrastructure.

```text
Ansible
   |
   ↓
Install Packages
Configure Servers
Deploy Applications
Start Services
```

### Complete Workflow

```text
Terraform
    |
    ↓
Create AWS Infrastructure
    |
    ↓
Create EC2 Instances
    |
    ↓
Get EC2 IP Address
    |
    ↓
Add IP to Ansible Inventory
    |
    ↓
Ansible
    |
    ↓
Configure EC2
    |
    ↓
Install Applications
    |
    ↓
Deploy Application
```

Example:

```text
Terraform → AWS EC2 → Ansible → Nginx → Application
```

---

# 16. Ansible + AWS

Ansible can also be used to manage AWS resources.

Examples include:

* EC2
* VPC
* Security Groups
* S3
* Load Balancers
* Route 53

Ansible can also configure applications running on AWS EC2 instances.

Example workflow:

```text
AWS
 |
 ├── EC2
 │    |
 │    └── Ansible configures server
 │
 ├── VPC
 │
 ├── Security Group
 │
 └── Load Balancer
```

---

# Complete Ansible Project Structure

```text
ansible-lab/
│
├── inventory
│
├── nginx.yml
├── variables.yml
├── conditional.yml
├── loops.yml
├── handlers.yml
├── template.yml
├── role.yml
├── vault.yml
├── secrets.yml
│
├── templates/
│   └── index.html.j2
│
└── nginx/
    ├── defaults/
    │   └── main.yml
    ├── handlers/
    │   └── main.yml
    ├── tasks/
    │   └── main.yml
    ├── templates/
    ├── files/
    ├── vars/
    │   └── main.yml
    ├── meta/
    │   └── main.yml
    └── README.md
```

---

# Useful Ansible Commands

## Check Ansible Version

```bash
ansible --version
```

## Ping All Hosts

```bash
ansible -i inventory all -m ping
```

## Check Uptime

```bash
ansible -i inventory all -m command -a "uptime"
```

## List Hosts

```bash
ansible -i inventory all --list-hosts
```

## Check Inventory

```bash
ansible-inventory -i inventory --list
```

## Run Playbook

```bash
ansible-playbook -i inventory playbook.yml
```

## Syntax Check

```bash
ansible-playbook -i inventory playbook.yml --syntax-check
```

## Check Mode

```bash
ansible-playbook -i inventory playbook.yml --check
```

## Gather Facts

```bash
ansible -i inventory all -m setup
```

## Create Role

```bash
ansible-galaxy init nginx
```

## Create Vault

```bash
ansible-vault create secrets.yml
```

## Edit Vault

```bash
ansible-vault edit secrets.yml
```

## View Vault

```bash
ansible-vault view secrets.yml
```

---

# Ansible Learning Roadmap

```text
1. Ansible Architecture
        ↓
2. Installation
        ↓
3. Inventory
        ↓
4. Ad-hoc Commands
        ↓
5. Playbooks
        ↓
6. Modules
        ↓
7. Variables
        ↓
8. Facts
        ↓
9. Conditionals
        ↓
10. Loops
        ↓
11. Handlers
        ↓
12. Templates (Jinja2)
        ↓
13. Roles
        ↓
14. Ansible Vault
        ↓
15. Terraform + Ansible
        ↓
16. Ansible + AWS
```

# Final Architecture

```text
                         AWS Cloud
                            |
                     ┌──────┴──────┐
                     │             │
                  EC2-1          EC2-2
               Control Node    Managed Node
                     │             │
                Ansible       No Ansible
                     │             │
                     └──── SSH ────┘
                            |
                       Configuration
                            |
                 ┌──────────┼──────────┐
                 ↓          ↓          ↓
               Nginx       Git       Docker
```

## Skills Covered

* Ansible Architecture
* Ansible Installation
* Inventory
* Ad-hoc Commands
* Playbooks
* Modules
* Variables
* Facts
* Conditionals
* Loops
* Handlers
* Jinja2 Templates
* Roles
* Ansible Vault
* Terraform + Ansible
* Ansible + AWS
* AWS EC2 Configuration
* Server Automation
* Application Deployment
