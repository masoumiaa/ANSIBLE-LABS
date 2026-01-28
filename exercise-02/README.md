# Exercise 02: Inventory Basics

## Learning Objectives

- Create and structure inventory files
- Organize hosts into groups
- Use inventory variables
- Understand different inventory formats

## Prerequisites

- Completed Exercise 01
- Basic understanding of YAML and INI formats

## Steps

### Step 1: Create an INI Inventory

Create a comprehensive inventory file:

```bash
cat > inventory.ini << 'EOF'
# Ungrouped hosts
standalone-server ansible_host=192.168.1.5

[webservers]
web1 ansible_host=192.168.1.10
web2 ansible_host=192.168.1.11

[databases]
db1 ansible_host=192.168.1.20
db2 ansible_host=192.168.1.21

[production:children]
webservers
databases

[local]
localhost ansible_connection=local
EOF
```

### Step 2: Test Your Inventory

List all hosts and groups:

```bash
# List all hosts
ansible all -i inventory.ini --list-hosts

# List specific group
ansible webservers -i inventory.ini --list-hosts

# List parent group (children)
ansible production -i inventory.ini --list-hosts
```

### Step 3: Create a YAML Inventory

Create the same inventory in YAML format:

```bash
cat > inventory.yml << 'EOF'
all:
  hosts:
    standalone-server:
      ansible_host: 192.168.1.5
  children:
    webservers:
      hosts:
        web1:
          ansible_host: 192.168.1.10
        web2:
          ansible_host: 192.168.1.11
    databases:
      hosts:
        db1:
          ansible_host: 192.168.1.20
        db2:
          ansible_host: 192.168.1.21
    production:
      children:
        webservers:
        databases:
    local:
      hosts:
        localhost:
          ansible_connection: local
EOF
```

### Step 4: Add Host Variables

Add variables to specific hosts:

```bash
cat > inventory-with-vars.ini << 'EOF'
[webservers]
web1 ansible_host=192.168.1.10 http_port=8080 max_connections=100
web2 ansible_host=192.168.1.11 http_port=8081 max_connections=200

[webservers:vars]
ansible_user=ubuntu
ansible_python_interpreter=/usr/bin/python3

[local]
localhost ansible_connection=local
EOF
```

### Step 5: Use Inventory Patterns

Test different inventory patterns:

```bash
# Ping all hosts
ansible all -i inventory.ini -m ping

# Ping specific host
ansible web1 -i inventory.ini -m ping

# Ping multiple groups
ansible 'webservers:databases' -i inventory.ini -m ping

# Ping all except a group
ansible 'all:!databases' -i inventory.ini -m ping

# Ping intersection of groups
ansible 'webservers:&production' -i inventory.ini -m ping

# Use wildcards
ansible 'web*' -i inventory.ini -m ping
```

### Step 6: View Host Variables

Check what variables are set for hosts:

```bash
# View all variables for a host
ansible localhost -i inventory.ini -m debug -a "var=hostvars"

# View specific group variables
ansible webservers -i inventory-with-vars.ini -m debug -a "var=ansible_user"
```

## Expected Results

### List Hosts Output

```
hosts (2):
  web1
  web2
```

### Ping with Variables

```json
web1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

## Inventory Structure Examples

### Simple Structure
```
[group_name]
hostname1
hostname2
```

### With Variables
```
[group_name]
hostname1 var1=value1 var2=value2

[group_name:vars]
common_var=common_value
```

### Nested Groups
```
[parent:children]
child_group1
child_group2
```

## Troubleshooting

### Problem: Host not found

**Solution:** Check spelling and group membership
```bash
ansible-inventory -i inventory.ini --list
```

### Problem: Variables not applying

**Solution:** Check variable precedence and syntax
```bash
ansible hostname -i inventory.ini -m debug -a "var=hostvars[inventory_hostname]"
```

## Additional Exercises

1. Create an inventory with 3 groups and 6 hosts
2. Add custom variables to each group
3. Create nested groups using `:children`
4. Test different inventory patterns
5. Convert an INI inventory to YAML format

## What You Learned

✅ Creating inventory files in INI and YAML formats  
✅ Organizing hosts into groups  
✅ Using host and group variables  
✅ Inventory patterns and selectors  
✅ Nested groups with `:children`  
✅ Viewing inventory structure  

## Next Steps

Move on to [Exercise 03: Ad-hoc Commands](../exercise-03/README.md) to learn more powerful one-line operations.
