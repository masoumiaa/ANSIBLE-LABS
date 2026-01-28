# Exercise 04: Your First Playbook

## Learning Objectives

- Understand playbook structure
- Write YAML playbooks
- Execute playbooks
- Use basic modules in playbooks

## Prerequisites

- Completed Exercise 03
- Understanding of YAML syntax

## Steps

### Step 1: Create Your First Playbook

Create a simple playbook that creates a file:

```bash
cat > first-playbook.yml << 'EOF'
---
- name: My First Playbook
  hosts: localhost
  connection: local
  
  tasks:
    - name: Create a directory
      file:
        path: /tmp/my-first-playbook
        state: directory
        mode: '0755'
    
    - name: Create a file with content
      copy:
        content: "This is my first Ansible playbook!\nCreated on: {{ ansible_date_time.date }}\n"
        dest: /tmp/my-first-playbook/info.txt
        mode: '0644'
    
    - name: Display success message
      debug:
        msg: "Playbook executed successfully! Check /tmp/my-first-playbook/info.txt"
EOF
```

### Step 2: Run the Playbook

Execute your first playbook:

```bash
# Run the playbook
ansible-playbook first-playbook.yml

# Run with increased verbosity
ansible-playbook first-playbook.yml -v

# Run in check mode (dry-run)
ansible-playbook first-playbook.yml --check
```

### Step 3: Verify the Results

Check what the playbook created:

```bash
# List the directory
ls -la /tmp/my-first-playbook/

# Read the file content
cat /tmp/my-first-playbook/info.txt
```

### Step 4: Create a Multi-Host Playbook

Create a playbook that works with inventory:

```bash
# Create inventory
cat > inventory.ini << 'EOF'
[local]
localhost ansible_connection=local
EOF

# Create multi-host playbook
cat > system-info.yml << 'EOF'
---
- name: Gather and Display System Information
  hosts: all
  gather_facts: yes
  
  tasks:
    - name: Display hostname
      debug:
        msg: "Hostname: {{ ansible_hostname }}"
    
    - name: Display OS information
      debug:
        msg: "OS: {{ ansible_distribution }} {{ ansible_distribution_version }}"
    
    - name: Display IP address
      debug:
        msg: "IP: {{ ansible_default_ipv4.address | default('N/A') }}"
    
    - name: Display memory info
      debug:
        msg: "Total Memory: {{ ansible_memtotal_mb }} MB"
EOF
```

### Step 5: Run Multi-Host Playbook

```bash
ansible-playbook -i inventory.ini system-info.yml
```

### Step 6: Create a Playbook with Multiple Plays

Create a playbook with different plays:

```bash
cat > multi-play.yml << 'EOF'
---
- name: First Play - File Operations
  hosts: localhost
  connection: local
  
  tasks:
    - name: Create test directory
      file:
        path: /tmp/ansible-multi-play
        state: directory
    
    - name: Create first file
      copy:
        content: "First play completed\n"
        dest: /tmp/ansible-multi-play/play1.txt

- name: Second Play - Information Display
  hosts: localhost
  connection: local
  gather_facts: yes
  
  tasks:
    - name: Display date
      debug:
        msg: "Current date: {{ ansible_date_time.date }}"
    
    - name: Create second file
      copy:
        content: "Second play completed\n"
        dest: /tmp/ansible-multi-play/play2.txt

- name: Third Play - Verification
  hosts: localhost
  connection: local
  
  tasks:
    - name: List created files
      command: ls -la /tmp/ansible-multi-play
      register: file_list
    
    - name: Display file list
      debug:
        var: file_list.stdout_lines
EOF
```

### Step 7: Execute and Test

```bash
# Run the multi-play playbook
ansible-playbook multi-play.yml

# Verify the results
ls -la /tmp/ansible-multi-play/
cat /tmp/ansible-multi-play/play1.txt
cat /tmp/ansible-multi-play/play2.txt
```

## Expected Results

### Successful Playbook Output

```
PLAY [My First Playbook] *****************************************************

TASK [Gathering Facts] *******************************************************
ok: [localhost]

TASK [Create a directory] ****************************************************
changed: [localhost]

TASK [Create a file with content] ********************************************
changed: [localhost]

TASK [Display success message] ***********************************************
ok: [localhost] => {
    "msg": "Playbook executed successfully! Check /tmp/my-first-playbook/info.txt"
}

PLAY RECAP *******************************************************************
localhost                  : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

## Playbook Structure

```yaml
---
- name: Play Name                    # Play description
  hosts: target_hosts                # Target hosts/groups
  connection: local                  # Connection type (optional)
  gather_facts: yes                  # Gather system facts (default: yes)
  become: yes                        # Use privilege escalation (optional)
  
  tasks:
    - name: Task Name                # Task description
      module_name:                   # Ansible module
        parameter1: value1           # Module parameters
        parameter2: value2
```

## Troubleshooting

### Problem: YAML syntax error

**Solution:** Check indentation (use spaces, not tabs)
```bash
# Validate YAML syntax
ansible-playbook playbook.yml --syntax-check
```

### Problem: Module not found

**Solution:** Check module name in documentation
```bash
ansible-doc module_name
```

### Problem: Task fails but want to continue

**Solution:** Add `ignore_errors: yes` to task
```yaml
- name: Task that might fail
  command: /some/command
  ignore_errors: yes
```

## Additional Exercises

1. Create a playbook that installs a package
2. Write a playbook with 5 different tasks
3. Create a playbook using variables
4. Add error handling to your playbook
5. Use the `register` keyword to capture task output

## What You Learned

✅ Basic playbook structure and YAML syntax  
✅ Writing and executing playbooks  
✅ Using multiple tasks in a play  
✅ Creating multi-play playbooks  
✅ Using check mode for dry-runs  
✅ Debugging with the debug module  
✅ Capturing and displaying task output  

## Next Steps

Move on to [Exercise 05: Multiple Tasks](../exercise-05/README.md) to learn advanced task management.
