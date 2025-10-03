# Ansible - IT Automation Platform

## Overview

Ansible is an open-source IT automation platform that automates cloud provisioning, configuration management, application deployment, intra-service orchestration, and many other IT needs. It uses a simple, agentless architecture with no special security requirements on the remote systems.

### Key Features
- **Agentless**: No agents to install on remote systems
- **Simple**: Uses YAML for configuration and playbooks
- **Powerful**: Manages complex multi-tier applications
- **Agentless Architecture**: Uses SSH for communication
- **Idempotent**: Ensures desired state is achieved
- **Extensible**: Custom modules and plugins
- **Agentless Security**: Uses existing SSH and sudo

## Core Concepts

### Ansible Architecture
- **Control Node**: Machine where Ansible is installed and run
- **Managed Nodes**: Machines that Ansible manages
- **Inventory**: List of managed nodes
- **Modules**: Tools that Ansible uses to perform tasks
- **Playbooks**: YAML files containing automation tasks
- **Roles**: Reusable collections of playbooks, templates, and files
- **Variables**: Values that can be used throughout playbooks

### Key Components
- **Ansible Core**: Main Ansible engine
- **Ansible Modules**: Reusable units of automation code
- **Ansible Playbooks**: Files containing automation instructions
- **Ansible Inventory**: Files defining managed systems
- **Ansible Roles**: Structured way to organize playbooks
- **Ansible Vault**: Encryption for sensitive data
- **Ansible Galaxy**: Repository for sharing roles

### Ansible Terminology
- **Playbook**: A file containing a series of tasks to be executed
- **Task**: A single unit of action in a playbook
- **Module**: A tool that Ansible uses to perform tasks
- **Inventory**: A file containing a list of managed hosts
- **Role**: A collection of playbooks, templates, and files
- **Handler**: A task that runs when another task changes something
- **Fact**: Information about a managed node

## Installation & Setup

### Prerequisites
- **Control Node**: Python 2.7 or Python 3.5+
- **Managed Nodes**: Python 2.6 or Python 3.5+ (for most modules)
- **Network**: SSH access to managed nodes
- **Authentication**: SSH keys or passwords for authentication

### Installation Methods

#### Using Package Manager (Ubuntu/Debian)
```bash
# Update package index
sudo apt update

# Install Ansible
sudo apt install ansible

# Verify installation
ansible --version
```

#### Using Package Manager (CentOS/RHEL)
```bash
# Install EPEL repository
sudo yum install epel-release

# Install Ansible
sudo yum install ansible

# Verify installation
ansible --version
```

#### Using pip
```bash
# Install Ansible using pip
pip install ansible

# Verify installation
ansible --version
```

#### Using Homebrew (macOS)
```bash
# Install Ansible
brew install ansible

# Verify installation
ansible --version
```

## Ansible Inventory

### Static Inventory File
```ini
# /etc/ansible/hosts
[web_servers]
web1.example.com
web2.example.com
web3.example.com

[database_servers]
db1.example.com
db2.example.com

[app_servers]
app1.example.com
app2.example.com

[all:vars]
ansible_user=ansible
ansible_ssh_private_key_file=~/.ssh/ansible_key

[web_servers:vars]
http_port=80
max_connections=100

[database_servers:vars]
mysql_port=3306
mysql_root_password=secure_password
```

### Dynamic Inventory
```yaml
# inventory.yml
plugin: aws_ec2
regions:
  - us-east-1
  - us-west-2
filters:
  instance-state-name: running
  tag:Environment:
    - production
keyed_groups:
  - key: tags.Environment
    prefix: env
  - key: tags.Role
    prefix: role
hostnames:
  - private-ip-address
compose:
  ansible_host: private_ip_address
```

### Inventory Groups
```ini
# Grouping hosts
[production]
web1.example.com
db1.example.com

[development]
web-dev.example.com
db-dev.example.com

[webservers:children]
production
development

[databases:children]
production
development
```

## Ansible Playbooks

### Basic Playbook Structure
```yaml
---
- name: Configure Web Server
  hosts: web_servers
  become: yes
  vars:
    http_port: 80
    max_connections: 100
  
  tasks:
    - name: Install Apache
      apt:
        name: apache2
        state: present
        update_cache: yes
    
    - name: Start Apache service
      service:
        name: apache2
        state: started
        enabled: yes
    
    - name: Copy website files
      copy:
        src: /path/to/files/
        dest: /var/www/html/
        owner: www-data
        group: www-data
        mode: '0644'
    
    - name: Configure Apache
      template:
        src: templates/apache2.conf.j2
        dest: /etc/apache2/apache2.conf
      notify:
        - Restart Apache
  
  handlers:
    - name: Restart Apache
      service:
        name: apache2
        state: restarted
```

### Playbook with Variables
```yaml
---
- name: Deploy Application
  hosts: app_servers
  become: yes
  vars_files:
    - vars/main.yml
    - vars/secrets.yml
  
  vars:
    app_version: "1.0.0"
    app_port: 8080
  
  tasks:
    - name: Create application user
      user:
        name: "{{ app_user }}"
        state: present
        shell: /bin/bash
    
    - name: Create application directory
      file:
        path: "{{ app_dir }}"
        state: directory
        owner: "{{ app_user }}"
        group: "{{ app_user }}"
        mode: '0755'
    
    - name: Download application
      get_url:
        url: "https://example.com/app-{{ app_version }}.tar.gz"
        dest: "/tmp/app-{{ app_version }}.tar.gz"
    
    - name: Extract application
      unarchive:
        src: "/tmp/app-{{ app_version }}.tar.gz"
        dest: "{{ app_dir }}"
        remote_src: yes
        owner: "{{ app_user }}"
        group: "{{ app_user }}"
    
    - name: Install application dependencies
      pip:
        requirements: "{{ app_dir }}/requirements.txt"
        virtualenv: "{{ app_dir }}/venv"
    
    - name: Create systemd service
      template:
        src: templates/app.service.j2
        dest: /etc/systemd/system/app.service
      notify:
        - Restart application
    
    - name: Start application service
      service:
        name: app
        state: started
        enabled: yes
  
  handlers:
    - name: Restart application
      service:
        name: app
        state: restarted
```

## Ansible Modules

### Common Modules

#### File Management
```yaml
- name: Create directory
  file:
    path: /opt/myapp
    state: directory
    owner: appuser
    group: appuser
    mode: '0755'

- name: Copy file
  copy:
    src: /local/path/file.txt
    dest: /remote/path/file.txt
    owner: appuser
    group: appuser
    mode: '0644'

- name: Download file
  get_url:
    url: https://example.com/file.txt
    dest: /tmp/file.txt
    mode: '0644'

- name: Create symbolic link
  file:
    src: /path/to/source
    dest: /path/to/link
    state: link
```

#### Package Management
```yaml
- name: Install package (Debian/Ubuntu)
  apt:
    name: nginx
    state: present
    update_cache: yes

- name: Install package (RedHat/CentOS)
  yum:
    name: nginx
    state: present

- name: Install multiple packages
  apt:
    name:
      - nginx
      - python3-pip
      - git
    state: present

- name: Remove package
  apt:
    name: old-package
    state: absent
```

#### Service Management
```yaml
- name: Start service
  service:
    name: nginx
    state: started
    enabled: yes

- name: Stop service
  service:
    name: nginx
    state: stopped

- name: Restart service
  service:
    name: nginx
    state: restarted

- name: Reload service
  service:
    name: nginx
    state: reloaded
```

#### User Management
```yaml
- name: Create user
  user:
    name: appuser
    state: present
    shell: /bin/bash
    groups: sudo
    append: yes

- name: Create user with password
  user:
    name: appuser
    state: present
    password: "{{ 'secure_password' | password_hash('sha512') }}"

- name: Create group
  group:
    name: appgroup
    state: present

- name: Add SSH key
  authorized_key:
    user: appuser
    state: present
    key: "{{ lookup('file', '/path/to/public_key.pub') }}"
```

#### System Management
```yaml
- name: Update system (Debian/Ubuntu)
  apt:
    upgrade: dist
    update_cache: yes

- name: Update system (RedHat/CentOS)
  yum:
    name: '*'
    state: latest

- name: Set hostname
  hostname:
    name: web-server-01

- name: Set timezone
  timezone:
    name: America/New_York
```

## Ansible Roles

### Role Structure
```
my_role/
├── defaults/
│   └── main.yml          # Default variables
├── files/                # Files to be copied
├── handlers/
│   └── main.yml          # Handlers
├── meta/
│   └── main.yml          # Role dependencies
├── tasks/
│   └── main.yml          # Task list
├── templates/            # Template files
├── tests/
│   ├── inventory
│   └── test.yml
└── vars/
    └── main.yml          # Role variables
```

### Example Role

#### tasks/main.yml
```yaml
---
- name: Install required packages
  apt:
    name: "{{ required_packages }}"
    state: present
    update_cache: yes

- name: Create application user
  user:
    name: "{{ app_user }}"
    state: present
    shell: /bin/bash

- name: Create application directory
  file:
    path: "{{ app_dir }}"
    state: directory
    owner: "{{ app_user }}"
    group: "{{ app_user }}"
    mode: '0755'

- name: Copy application files
  copy:
    src: app/
    dest: "{{ app_dir }}"
    owner: "{{ app_user }}"
    group: "{{ app_user }}"
    mode: '0644'

- name: Template configuration file
  template:
    src: config.j2
    dest: "{{ app_config_file }}"
    owner: "{{ app_user }}"
    group: "{{ app_user }}"
    mode: '0644'
  notify: Restart application

- name: Create systemd service
  template:
    src: app.service.j2
    dest: /etc/systemd/system/app.service
  notify: Restart application

- name: Start application service
  service:
    name: app
    state: started
    enabled: yes
```

#### defaults/main.yml
```yaml
---
app_user: appuser
app_dir: /opt/app
app_config_file: /opt/app/config.yml
required_packages:
  - python3
  - python3-pip
  - git
```

#### handlers/main.yml
```yaml
---
- name: Restart application
  service:
    name: app
    state: restarted
```

#### templates/app.service.j2
```ini
[Unit]
Description=My Application
After=network.target

[Service]
Type=simple
User={{ app_user }}
WorkingDirectory={{ app_dir }}
ExecStart={{ app_dir }}/venv/bin/python {{ app_dir }}/app.py
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

## Ansible Vault

### Encrypting Files
```bash
# Encrypt a file
ansible-vault encrypt secrets.yml

# Encrypt a file with specific password file
ansible-vault encrypt --vault-password-file ~/.ansible_vault_pass secrets.yml

# Create new encrypted file
ansible-vault create new_secrets.yml

# Edit encrypted file
ansible-vault edit secrets.yml

# View encrypted file
ansible-vault view secrets.yml

# Decrypt file
ansible-vault decrypt secrets.yml

# Change password
ansible-vault rekey secrets.yml
```

### Using Vault in Playbooks
```yaml
---
- name: Deploy Application
  hosts: app_servers
  become: yes
  vars_files:
    - vars/secrets.yml
  
  tasks:
    - name: Use secret variable
      debug:
        msg: "Database password is {{ db_password }}"
```

## Ansible Commands

### Basic Commands
```bash
# Test connectivity to all hosts
ansible all -m ping

# Test connectivity to specific group
ansible webservers -m ping

# Run command on all hosts
ansible all -a "hostname"

# Run command with sudo
ansible all -a "apt update" --become

# Check disk space
ansible all -a "df -h"

# Check memory usage
ansible all -a "free -m"

# List all hosts
ansible-inventory --list

# Check host variables
ansible-inventory --host web1.example.com
```

### Playbook Commands
```bash
# Run playbook
ansible-playbook playbook.yml

# Run playbook with specific inventory
ansible-playbook -i inventory.yml playbook.yml

# Run playbook with specific tags
ansible-playbook --tags "install,configure" playbook.yml

# Run playbook skipping specific tags
ansible-playbook --skip-tags "test" playbook.yml

# Run playbook with extra variables
ansible-playbook --extra-vars "app_version=1.0.0" playbook.yml

# Run playbook with vault password
ansible-playbook --vault-password-file ~/.ansible_vault_pass playbook.yml

# Run playbook in check mode (dry run)
ansible-playbook --check playbook.yml

# Run playbook with verbose output
ansible-playbook -v playbook.yml
ansible-playbook -vv playbook.yml
ansible-playbook -vvv playbook.yml
```

### Advanced Commands
```bash
# Run playbook with limit
ansible-playbook --limit webservers playbook.yml

# Run playbook with specific user
ansible-playbook --user ansible --become playbook.yml

# Run playbook with specific SSH key
ansible-playbook --private-key ~/.ssh/ansible_key playbook.yml

# Run playbook with specific forks
ansible-playbook --forks 10 playbook.yml

# Run playbook with specific timeout
ansible-playbook --timeout 60 playbook.yml

# Run playbook with specific inventory file
ansible-playbook -i production_inventory playbook.yml

# Run playbook with specific vault ID
ansible-playbook --vault-id @prompt playbook.yml
```

## Ansible Galaxy

### Using Galaxy
```bash
# Search for roles
ansible-galaxy search nginx

# Install role
ansible-galaxy install geerlingguy.nginx

# Install role from specific version
ansible-galaxy install geerlingguy.nginx,2.7.0

# Install role from GitHub
ansible-galaxy install git+https://github.com/geerlingguy/ansible-role-nginx.git

# Install multiple roles from requirements file
ansible-galaxy install -r requirements.yml

# List installed roles
ansible-galaxy list

# Remove role
ansible-galaxy remove geerlingguy.nginx

# Initialize role
ansible-galaxy init my_role
```

### requirements.yml
```yaml
---
- src: geerlingguy.nginx
  version: 2.7.0
- src: geerlingguy.mysql
  version: 3.2.0
- src: geerlingguy.redis
  version: 1.5.0
- src: https://github.com/geerlingguy/ansible-role-postgresql
  version: master
  name: postgresql
```

## Ansible Templates

### Template Example
```jinja2
# config.j2
# {{ ansible_managed }}

server {
    listen {{ http_port }};
    server_name {{ server_name }};
    
    root {{ web_root }};
    index index.html index.htm;
    
    location / {
        try_files $uri $uri/ =404;
    }
    
    location /api {
        proxy_pass http://localhost:{{ app_port }};
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    
    access_log /var/log/nginx/{{ server_name }}.access.log;
    error_log /var/log/nginx/{{ server_name }}.error.log;
}
```

### Template Filters
```yaml
- name: Create configuration file
  template:
    src: config.j2
    dest: /etc/app/config.yml
  vars:
    db_password: "{{ vault_db_password | b64decode }}"
    app_version: "{{ app_version | default('1.0.0') }}"
    server_list: "{{ servers | join(',') }}"
```

## Ansible Conditionals

### Conditional Tasks
```yaml
- name: Install Apache on Debian
  apt:
    name: apache2
    state: present
  when: ansible_os_family == "Debian"

- name: Install Apache on RedHat
  yum:
    name: httpd
    state: present
  when: ansible_os_family == "RedHat"

- name: Configure firewall
  ufw:
    state: enabled
    rule: allow
    port: "{{ http_port }}"
    proto: tcp
  when: enable_firewall | default(true)

- name: Start service
  service:
    name: "{{ service_name }}"
    state: started
    enabled: yes
  when: service_enabled
```

### Complex Conditions
```yaml
- name: Install package
  apt:
    name: "{{ package_name }}"
    state: present
  when: 
    - ansible_os_family == "Debian"
    - package_name is defined
    - package_version is version('1.0.0', '>=')
    - not skip_install | default(false)
```

## Ansible Loops

### Standard Loops
```yaml
- name: Create multiple users
  user:
    name: "{{ item }}"
    state: present
  loop:
    - user1
    - user2
    - user3

- name: Install multiple packages
  apt:
    name: "{{ item }}"
    state: present
  loop:
    - nginx
    - python3-pip
    - git

- name: Create multiple directories
  file:
    path: "/opt/{{ item }}"
    state: directory
    mode: '0755'
  loop:
    - app1
    - app2
    - app3
```

### Dictionary Loops
```yaml
- name: Create users with home directories
  user:
    name: "{{ item.key }}"
    home: "/home/{{ item.value.home_dir }}"
    shell: "{{ item.value.shell }}"
  loop: "{{ users | dict2items }}"
  vars:
    users:
      user1:
        home_dir: user1
        shell: /bin/bash
      user2:
        home_dir: user2
        shell: /bin/zsh
```

### Nested Loops
```yaml
- name: Create multiple files in multiple directories
  file:
    path: "/opt/{{ item.0 }}/{{ item.1 }}"
    state: touch
  loop: "{{ directories | product(files) | list }}"
  vars:
    directories:
      - app1
      - app2
    files:
      - config.yml
      - log.txt
```

## Ansible Error Handling

### Error Handling Examples
```yaml
- name: Try to install package
  apt:
    name: non-existent-package
    state: present
  ignore_errors: yes

- name: Continue on failure
  block:
    - name: Try to start service
      service:
        name: non-existent-service
        state: started
  rescue:
    - name: Handle failure
      debug:
        msg: "Service not found, continuing..."
  always:
    - name: Always run
      debug:
        msg: "This always runs"

- name: Fail under specific condition
  fail:
    msg: "This task must fail"
  when: force_fail | default(false)
```

## Interview Questions

### Beginner Level
1. **What is Ansible?**
   - Ansible is an open-source IT automation platform that automates cloud provisioning, configuration management, and application deployment

2. **What are the key features of Ansible?**
   - Agentless architecture, simple YAML configuration, idempotent operations, and no special security requirements

3. **What is an Ansible playbook?**
   - A playbook is a YAML file containing a series of tasks to be executed on managed nodes

4. **What is an Ansible inventory?**
   - An inventory is a file containing a list of managed nodes and their groupings

### Intermediate Level
1. **What is the difference between Ansible and other configuration management tools?**
   - Ansible is agentless, uses SSH for communication, and has a simpler learning curve compared to tools like Puppet or Chef

2. **What are Ansible modules?**
   - Modules are reusable units of automation code that perform specific tasks like installing packages or managing files

3. **What is idempotency in Ansible?**
   - Idempotency means that running a task multiple times will result in the same state as running it once

4. **What are Ansible roles?**
   - Roles are structured collections of playbooks, templates, files, and variables that can be reused across different playbooks

### Advanced Level
1. **How does Ansible handle security?**
   - Ansible uses SSH for communication, supports SSH key authentication, and provides Ansible Vault for encrypting sensitive data

2. **What is Ansible Galaxy?**
   - Ansible Galaxy is a repository for sharing and downloading Ansible roles created by the community

3. **How does Ansible handle dynamic inventory?**
   - Dynamic inventory scripts can generate inventory from external sources like cloud providers, CMDB systems, or other APIs

4. **What are Ansible callbacks?**
   - Callbacks are pieces of code that get called when specific events occur during playbook execution, allowing for custom logging and notifications

## Best Practices

### Playbook Design
- **Idempotent Operations**: Ensure tasks can be run multiple times safely
- **Modular Design**: Use roles and includes for modularity
- **Variable Management**: Use variables for configuration values
- **Error Handling**: Implement proper error handling and recovery
- **Documentation**: Document playbooks and roles

### Security
- **SSH Keys**: Use SSH key authentication instead of passwords
- **Ansible Vault**: Encrypt sensitive data with Ansible Vault
- **Least Privilege**: Run with minimal required privileges
- **Secure Inventory**: Protect inventory files with proper permissions
- **Network Security**: Use secure network connections

### Performance
- **Forks**: Adjust fork count for parallel execution
- **Fact Caching**: Cache facts to improve performance
- **Connection Pipelining**: Enable connection pipelining
- **Strategy Plugins**: Use appropriate strategy plugins
- **Selective Execution**: Use tags and limits for selective execution

## Troubleshooting

### Common Issues
```bash
# Test connectivity
ansible all -m ping

# Check SSH connectivity
ansible all -m raw -a "ssh -V"

# Check Python version
ansible all -m raw -a "python --version"

# Check disk space
ansible all -a "df -h"

# Check memory usage
ansible all -a "free -m"

# Check service status
ansible all -a "systemctl status nginx"

# Check log files
ansible all -a "tail -n 50 /var/log/syslog"
```

### Debugging Playbooks
```bash
# Run playbook with verbose output
ansible-playbook -vvv playbook.yml

# Run playbook in check mode
ansible-playbook --check playbook.yml

# Run playbook with specific tags
ansible-playbook --tags "debug" playbook.yml

# Check syntax
ansible-playbook --syntax-check playbook.yml

# List tasks
ansible-playbook --list-tasks playbook.yml

# List hosts
ansible-playbook --list-hosts playbook.yml
```

## Resources

### Official Documentation
- [Ansible Documentation](https://docs.ansible.com/)
- [Ansible User Guide](https://docs.ansible.com/ansible/latest/user_guide/)
- [Ansible Module Index](https://docs.ansible.com/ansible/latest/modules/modules_by_category.html)

### Learning Resources
- [Ansible Tutorial](https://docs.ansible.com/ansible/latest/user_guide/tutorial.html)
- [Ansible Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)
- [Ansible Galaxy](https://galaxy.ansible.com/)

### Community
- [Ansible Community](https://www.ansible.com/community)
- [Ansible Forum](https://forum.ansible.com/)
- [Ansible GitHub](https://github.com/ansible/ansible)