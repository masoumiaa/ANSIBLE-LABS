# Ansible Labs for SRC - M1 - ESIG

Welcome to the Ansible Labs! This repository contains 20 hands-on exercises designed to help you guys learn Ansible from the ground up.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Lab Environment Setup](#lab-environment-setup)
- [Exercises](#exercises)
- [Getting Help](#getting-help)

## Prerequisites

Before starting these labs, you should have:

- Basic understanding of Linux command line
- Familiarity with YAML syntax (helpful but not required)
- A computer running Linux, macOS, or Windows with WSL2
- SSH access to at least one remote machine (or use local connection)
- Basic networking knowledge

### System Requirements

- **RAM**: Minimum 2GB (4GB recommended)
- **Disk Space**: At least 5GB free
- **OS**: Ubuntu 20.04+, CentOS 7+, macOS 10.14+, or Windows 10+ with WSL2 :D (WSL qui font exploser les ordis, du coup, Linux ;)

## Installation

### Linux (Ubuntu/Debian)

```bash
# Update package index
sudo apt update

# Install Python and pip
sudo apt install python3 python3-pip -y

# Install Ansible
sudo apt install ansible -y

# Verify installation
ansible --version
```

### Linux (RHEL/CentOS/Fedora)

```bash
# Install EPEL repository (CentOS/RHEL)
sudo yum install epel-release -y

# Install Ansible
sudo yum install ansible -y

# Verify installation
ansible --version
```

### macOS

```bash
# Install Homebrew if not already installed
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install Ansible
brew install ansible

# Verify installation
ansible --version
```

### Windows (WSL2)

```bash
# Install WSL2 with Ubuntu from PowerShell (Admin)
wsl --install

# Inside WSL2 Ubuntu terminal:
sudo apt update
sudo apt install ansible -y

# Verify installation
ansible --version
```

### Alternative: Using pip

```bash
# Install using pip (all platforms)
pip3 install ansible

# Verify installation
ansible --version
```

## Lab Environment Setup

### Option 1: Local Setup (Recommended for Beginners)

Test Ansible on your local machine first:

```bash
# Create a working directory
mkdir -p ~/ansible-labs
cd ~/ansible-labs

# Test local connection
ansible localhost -m ping
```

### Option 2: Virtual Machines

Use VirtualBox or VMware to create test VMs:

1. Create 2-3 Ubuntu/CentOS VMs
2. Configure SSH access
3. Note down IP addresses

### Option 3: Docker Containers

```bash
# Run SSH-enabled containers for testing
docker run -d --name ansible-node1 -p 2221:22 rastasheep/ubuntu-sshd
docker run -d --name ansible-node2 -p 2222:22 rastasheep/ubuntu-sshd
```

### Configure SSH Keys (Recommended)

```bash
# Generate SSH key pair if you don't have one
ssh-keygen -t rsa -b 4096

# Copy public key to remote hosts
ssh-copy-id user@remote-host-ip

# Test SSH connection
ssh user@remote-host-ip
```

### Create Ansible Configuration File

```bash
# Create ansible.cfg in your working directory
cat > ansible.cfg << EOF
[defaults]
inventory = inventory.ini
host_key_checking = False
retry_files_enabled = False
interpreter_python = auto_silent

[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = False
EOF
```

## Exercises

Each exercise is self-contained in its own directory with a dedicated README file containing:
- Learning objectives
- Step-by-step instructions
- Expected results
- Troubleshooting tips

### Basic Concepts (1-5)

1. [Exercise 01: Your First Ansible Command](exercise-01/README.md) - Ad-hoc commands and ping module
2. [Exercise 02: Inventory Basics](exercise-02/README.md) - Creating and managing inventory files
3. [Exercise 03: Ad-hoc Commands](exercise-03/README.md) - Using modules without playbooks
4. [Exercise 04: Your First Playbook](exercise-04/README.md) - Creating a simple playbook
5. [Exercise 05: Multiple Tasks](exercise-05/README.md) - Combining multiple tasks in one playbook

### Playbooks and Tasks (6-10)

6. [Exercise 06: Working with Handlers](exercise-06/README.md) - Using handlers for service management
7. [Exercise 07: Conditional Execution](exercise-07/README.md) - Using when statements
8. [Exercise 08: Loops in Playbooks](exercise-08/README.md) - Iterating over lists
9. [Exercise 09: File Management](exercise-09/README.md) - Copy, template, and file modules
10. [Exercise 10: Package Management](exercise-10/README.md) - Installing and managing packages

### Variables and Templates (11-15)

11. [Exercise 11: Using Variables](exercise-11/README.md) - Defining and using variables
12. [Exercise 12: Variable Files](exercise-12/README.md) - External variable files
13. [Exercise 13: Jinja2 Templates](exercise-13/README.md) - Creating configuration templates
14. [Exercise 14: Facts and Filters](exercise-14/README.md) - Gathering and using system facts
15. [Exercise 15: Vault Secrets](exercise-15/README.md) - Encrypting sensitive data

### Advanced Topics (16-20)

16. [Exercise 16: Roles Basics](exercise-16/README.md) - Creating and using roles
17. [Exercise 17: Error Handling](exercise-17/README.md) - Managing failures and blocks
18. [Exercise 18: Tags and Task Control](exercise-18/README.md) - Using tags for selective execution
19. [Exercise 19: Dynamic Inventory](exercise-19/README.md) - Using dynamic inventory sources
20. [Exercise 20: Full Web Server Deployment](exercise-20/README.md) - Complete practical project

## Getting Help

### Documentation

- [Official Ansible Documentation](https://docs.ansible.com/)
- [Ansible Module Index](https://docs.ansible.com/ansible/latest/collections/index_module.html)
- [Ansible Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)

### Common Issues

#### SSH Connection Issues

```bash
# Test SSH connectivity
ansible all -m ping -i inventory.ini

# Use password authentication if keys don't work
ansible all -m ping -i inventory.ini --ask-pass
```

#### Permission Issues

```bash
# Use become for privilege escalation
ansible all -m shell -a "whoami" --become --ask-become-pass
```

#### Python Interpreter Issues

```bash
# Specify Python interpreter
ansible all -m ping -e "ansible_python_interpreter=/usr/bin/python3"
```

### Tips for Success

1. **Start Simple**: Begin with local execution before moving to remote hosts
2. **Read Error Messages**: Ansible provides detailed error messages
3. **Use --check Mode**: Test playbooks without making changes (`ansible-playbook playbook.yml --check`)
4. **Increase Verbosity**: Use `-v`, `-vv`, or `-vvv` for more detailed output
5. **Test in Isolation**: Test each exercise independently before moving on

## 👨‍🏫 Credits

Course Author: Dr. Amine SOUMIAA  
Program: Master SRC (M1)
Institution: ESGI – École Supérieure de Génie Informatique  
Website: https://www.esgi.fr/  

© 2026 – All rights reserved for academic use.

---

**Happy Learning! 🚀**