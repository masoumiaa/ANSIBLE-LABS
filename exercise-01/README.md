# Exercise 01: Your First Ansible Command

## Learning Objectives

- Understand Ansible ad-hoc commands
- Use the `ping` module to test connectivity
- Learn basic Ansible command syntax

## Prerequisites

- Ansible installed (verify with `ansible --version`)
- Access to localhost or a remote server

## Steps

### Step 1: Test Local Connection

Run your first Ansible command to ping localhost:

```bash
ansible localhost -m ping
```

**Explanation:**
- `localhost` - target host
- `-m ping` - use the ping module (tests Python and connectivity)

### Step 2: Create a Basic Inventory File

Create an inventory file to manage hosts:

```bash
# Create inventory.ini
cat > inventory.ini << EOF
[local]
localhost ansible_connection=local

[webservers]
# Add your remote servers here if available
# server1 ansible_host=192.168.1.10 ansible_user=ubuntu
EOF
```

### Step 3: Ping Using Inventory

Test connectivity using the inventory file:

```bash
# Ping all hosts
ansible all -i inventory.ini -m ping

# Ping specific group
ansible local -i inventory.ini -m ping
```

### Step 4: Increase Verbosity

Run with verbose output to see what's happening:

```bash
ansible localhost -m ping -v
ansible localhost -m ping -vv
ansible localhost -m ping -vvv
```

### Step 5: Check Ansible Configuration

Verify your Ansible setup:

```bash
# Show ansible version
ansible --version

# List all hosts in inventory
ansible all -i inventory.ini --list-hosts
```

## Expected Results

### Successful Ping Output

```json
localhost | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

### Key Points

- **SUCCESS** - Connection was successful
- **"changed": false** - No changes were made to the system
- **"ping": "pong"** - Module executed successfully

## Troubleshooting

### Problem: "command not found"

**Solution:** Ansible is not installed or not in PATH
```bash
# Verify installation
which ansible
# Install if needed
pip3 install ansible
```

### Problem: "Permission denied"

**Solution:** SSH key or password authentication issue
```bash
# Use password authentication
ansible localhost -m ping --ask-pass
```

### Problem: Python interpreter not found

**Solution:** Specify Python interpreter
```bash
ansible localhost -m ping -e "ansible_python_interpreter=/usr/bin/python3"
```

## Additional Exercises

1. Try pinging with different verbosity levels (-v, -vv, -vvv)
2. Add a remote host to your inventory and ping it
3. Explore the JSON output format using `-o` flag

## What You Learned

✅ How to execute ad-hoc Ansible commands  
✅ The purpose of the ping module  
✅ Basic Ansible command syntax  
✅ How to use verbosity for debugging  
✅ Understanding Ansible output format  

## Next Steps

Move on to [Exercise 02: Inventory Basics](../exercise-02/README.md) to learn more about managing multiple hosts.
