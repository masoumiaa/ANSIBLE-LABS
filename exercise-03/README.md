# Exercise 03: Ad-hoc Commands

## Learning Objectives

- Execute quick tasks without playbooks
- Use common Ansible modules
- Understand module parameters
- Perform system administration tasks

## Prerequisites

- Completed Exercise 02
- Access to a test system (localhost or remote)

## Steps

### Step 1: Setup Test Inventory

Create a simple inventory:

```bash
cat > inventory.ini << 'EOF'
[local]
localhost ansible_connection=local
EOF
```

### Step 2: System Information Commands

Gather system information using ad-hoc commands:

```bash
# Check disk space
ansible localhost -i inventory.ini -m shell -a "df -h"

# Check memory usage
ansible localhost -i inventory.ini -m shell -a "free -m"

# Check uptime
ansible localhost -i inventory.ini -m command -a "uptime"

# Gather all system facts
ansible localhost -i inventory.ini -m setup
```

### Step 3: File Operations

Manage files using modules:

```bash
# Create a directory
ansible localhost -i inventory.ini -m file -a "path=/tmp/ansible-test state=directory mode=0755"

# Create a file
ansible localhost -i inventory.ini -m file -a "path=/tmp/ansible-test/test.txt state=touch mode=0644"

# Copy a file
ansible localhost -i inventory.ini -m copy -a "content='Hello Ansible\n' dest=/tmp/ansible-test/hello.txt"

# Check if file exists
ansible localhost -i inventory.ini -m stat -a "path=/tmp/ansible-test/hello.txt"

# Read file content
ansible localhost -i inventory.ini -m slurp -a "src=/tmp/ansible-test/hello.txt"
```

### Step 4: User and Group Management

Manage users (may require sudo):

```bash
# Check current user
ansible localhost -i inventory.ini -m command -a "whoami"

# Create a group
ansible localhost -i inventory.ini -m group -a "name=testgroup state=present" --become

# Create a user
ansible localhost -i inventory.ini -m user -a "name=testuser group=testgroup state=present" --become

# Remove user
ansible localhost -i inventory.ini -m user -a "name=testuser state=absent remove=yes" --become

# Remove group
ansible localhost -i inventory.ini -m group -a "name=testgroup state=absent" --become
```

### Step 5: Package Management

Install and remove packages:

```bash
# Update package cache (Debian/Ubuntu)
ansible localhost -i inventory.ini -m apt -a "update_cache=yes" --become

# Install a package
ansible localhost -i inventory.ini -m apt -a "name=curl state=present" --become

# Check if package is installed
ansible localhost -i inventory.ini -m command -a "which curl"

# Remove a package
ansible localhost -i inventory.ini -m apt -a "name=curl state=absent" --become
```

### Step 6: Service Management

Manage services:

```bash
# Check service status
ansible localhost -i inventory.ini -m service_facts

# Start a service (example: ssh)
ansible localhost -i inventory.ini -m service -a "name=ssh state=started" --become

# Check service status
ansible localhost -i inventory.ini -m systemd -a "name=ssh" --become
```

### Step 7: Variable and Debug Module

Use debug module for testing:

```bash
# Print a message
ansible localhost -i inventory.ini -m debug -a "msg='Hello from Ansible'"

# Print a variable
ansible localhost -i inventory.ini -m debug -a "var=ansible_date_time" -m setup

# Print inventory hostname
ansible localhost -i inventory.ini -m debug -a "var=inventory_hostname"
```

## Expected Results

### Successful Command Output

```
localhost | CHANGED | rc=0 >>
Hello Ansible
```

### File Creation

```json
localhost | SUCCESS => {
    "changed": true,
    "dest": "/tmp/ansible-test/hello.txt",
    "gid": 1000,
    "mode": "0644",
    "owner": "user",
    "size": 14,
    "state": "file"
}
```

## Common Modules Reference

| Module | Purpose | Example |
|--------|---------|---------|
| `ping` | Test connectivity | `-m ping` |
| `command` | Run command (no shell) | `-m command -a "ls -la"` |
| `shell` | Run shell command | `-m shell -a "echo $HOME"` |
| `file` | Manage files/directories | `-m file -a "path=/tmp/test state=directory"` |
| `copy` | Copy files | `-m copy -a "src=file.txt dest=/tmp/"` |
| `apt/yum` | Package management | `-m apt -a "name=nginx state=present"` |
| `service` | Service management | `-m service -a "name=nginx state=started"` |
| `user` | User management | `-m user -a "name=john state=present"` |
| `setup` | Gather facts | `-m setup` |
| `debug` | Print messages | `-m debug -a "msg='Hello'"` |

## Troubleshooting

### Problem: "Permission denied"

**Solution:** Use `--become` for privilege escalation
```bash
ansible localhost -i inventory.ini -m apt -a "name=curl state=present" --become
```

### Problem: Module not found

**Solution:** Check module name and Ansible version
```bash
ansible-doc -l | grep module_name
```

### Problem: Command vs Shell module

**Solution:** Use `shell` for pipes, redirects, environment variables
```bash
# Use command for simple commands
ansible localhost -m command -a "ls /tmp"

# Use shell for complex commands
ansible localhost -m shell -a "ls /tmp | grep test"
```

## Additional Exercises

1. Create a directory structure with multiple nested folders
2. Install 3 different packages in one command
3. Create 2 users and add them to a common group
4. Use the `setup` module to find your system's IP address
5. Copy multiple files using a loop pattern

## What You Learned

✅ Executing ad-hoc commands for quick tasks  
✅ Using common Ansible modules (file, copy, apt, user, etc.)  
✅ Difference between command and shell modules  
✅ Using --become for privilege escalation  
✅ Managing files, packages, users, and services  
✅ Debugging with the debug module  

## Next Steps

Move on to [Exercise 04: Your First Playbook](../exercise-04/README.md) to start writing reusable automation scripts.
