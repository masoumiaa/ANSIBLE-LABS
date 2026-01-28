# Exercise 08: Loops in Playbooks

## Learning Objectives

- Use loops to iterate over lists
- Loop over dictionaries
- Use different loop constructs
- Combine loops with conditions

## Prerequisites

- Completed Exercise 07
- Understanding of lists and dictionaries

## Steps

### Step 1: Basic Loop with List

Create a playbook with simple loops:

```bash
cat > basic-loops.yml << 'EOF'
---
- name: Basic Loops
  hosts: localhost
  connection: local
  
  tasks:
    - name: Create multiple directories
      file:
        path: "/tmp/loop-test/{{ item }}"
        state: directory
      loop:
        - dir1
        - dir2
        - dir3
        - dir4
        - dir5
    
    - name: Create multiple files
      copy:
        content: "File created by loop: {{ item }}\n"
        dest: "/tmp/loop-test/{{ item }}.txt"
      loop:
        - file1
        - file2
        - file3
    
    - name: Display loop items
      debug:
        msg: "Processing item: {{ item }}"
      loop:
        - apple
        - banana
        - cherry
        - date
EOF
```

### Step 2: Run Basic Loop Playbook

```bash
ansible-playbook basic-loops.yml

# Verify results
ls -la /tmp/loop-test/
```

### Step 3: Loop with Variables

Create a playbook using variables in loops:

```bash
cat > loop-variables.yml << 'EOF'
---
- name: Loops with Variables
  hosts: localhost
  connection: local
  vars:
    users:
      - alice
      - bob
      - charlie
    packages:
      - curl
      - wget
      - git
    ports:
      - 8080
      - 8081
      - 8082
  
  tasks:
    - name: Display users
      debug:
        msg: "User: {{ item }}"
      loop: "{{ users }}"
    
    - name: Display packages to install
      debug:
        msg: "Would install package: {{ item }}"
      loop: "{{ packages }}"
    
    - name: Create config for each port
      copy:
        content: |
          # Configuration for port {{ item }}
          port={{ item }}
          enabled=true
        dest: "/tmp/loop-test/port-{{ item }}.conf"
      loop: "{{ ports }}"
EOF
```

### Step 4: Test Variable Loops

```bash
ansible-playbook loop-variables.yml
```

### Step 5: Loop over Dictionaries

Create a playbook that loops over dictionary data:

```bash
cat > dict-loops.yml << 'EOF'
---
- name: Loop Over Dictionaries
  hosts: localhost
  connection: local
  vars:
    users:
      - name: alice
        uid: 1001
        shell: /bin/bash
      - name: bob
        uid: 1002
        shell: /bin/zsh
      - name: charlie
        uid: 1003
        shell: /bin/bash
    
    applications:
      - app: nginx
        port: 80
        enabled: true
      - app: postgresql
        port: 5432
        enabled: true
      - app: redis
        port: 6379
        enabled: false
  
  tasks:
    - name: Display user information
      debug:
        msg: "User {{ item.name }} has UID {{ item.uid }} and shell {{ item.shell }}"
      loop: "{{ users }}"
    
    - name: Create user info files
      copy:
        content: |
          Username: {{ item.name }}
          UID: {{ item.uid }}
          Shell: {{ item.shell }}
        dest: "/tmp/loop-test/user-{{ item.name }}.txt"
      loop: "{{ users }}"
    
    - name: Display enabled applications
      debug:
        msg: "{{ item.app }} runs on port {{ item.port }}"
      loop: "{{ applications }}"
      when: item.enabled == true
    
    - name: Create app configs
      copy:
        content: |
          [{{ item.app }}]
          port={{ item.port }}
          enabled={{ item.enabled }}
        dest: "/tmp/loop-test/{{ item.app }}.conf"
      loop: "{{ applications }}"
EOF
```

### Step 6: Run Dictionary Loop Playbook

```bash
ansible-playbook dict-loops.yml

# Check created files
cat /tmp/loop-test/user-alice.txt
cat /tmp/loop-test/nginx.conf
```

### Step 7: Loop Control Features

Create a playbook using loop control:

```bash
cat > loop-control.yml << 'EOF'
---
- name: Loop Control Features
  hosts: localhost
  connection: local
  
  tasks:
    - name: Loop with custom variable name
      debug:
        msg: "Processing user: {{ user }}"
      loop:
        - alice
        - bob
        - charlie
      loop_control:
        loop_var: user
    
    - name: Loop with index
      debug:
        msg: "Item {{ idx }}: {{ item }}"
      loop:
        - first
        - second
        - third
        - fourth
      loop_control:
        index_var: idx
    
    - name: Loop with pause
      debug:
        msg: "Processing: {{ item }}"
      loop:
        - task1
        - task2
        - task3
      loop_control:
        pause: 2
    
    - name: Loop with label (cleaner output)
      copy:
        content: "Config for {{ item.name }}\n"
        dest: "/tmp/loop-test/{{ item.name }}-config.txt"
      loop:
        - name: service1
          port: 8001
          description: "First service"
        - name: service2
          port: 8002
          description: "Second service"
      loop_control:
        label: "{{ item.name }}"
EOF
```

### Step 8: Test Loop Control

```bash
ansible-playbook loop-control.yml
```

### Step 9: Nested Loops

Create a playbook with nested loops:

```bash
cat > nested-loops.yml << 'EOF'
---
- name: Nested Loops
  hosts: localhost
  connection: local
  vars:
    environments:
      - dev
      - staging
      - prod
    
    services:
      - web
      - api
      - db
  
  tasks:
    - name: Create directory structure (nested loops)
      file:
        path: "/tmp/loop-test/{{ item.0 }}/{{ item.1 }}"
        state: directory
      loop: "{{ environments | product(services) | list }}"
    
    - name: Display combinations
      debug:
        msg: "Environment: {{ item.0 }}, Service: {{ item.1 }}"
      loop: "{{ environments | product(services) | list }}"
      loop_control:
        label: "{{ item.0 }}-{{ item.1 }}"
    
    - name: Alternative nested loop syntax
      debug:
        msg: "Creating config for {{ env_item }}/{{ service_item }}"
      loop: "{{ environments }}"
      loop_control:
        loop_var: env_item
      vars:
        inner_loop: "{{ services }}"
      when: false  # Disabled for demonstration
EOF
```

### Step 10: Run Nested Loops

```bash
ansible-playbook nested-loops.yml

# Verify directory structure
tree /tmp/loop-test/  # or use ls -R
```

## Expected Results

### Loop Execution Output

```
TASK [Create multiple directories] *******************************************
changed: [localhost] => (item=dir1)
changed: [localhost] => (item=dir2)
changed: [localhost] => (item=dir3)
changed: [localhost] => (item=dir4)
changed: [localhost] => (item=dir5)
```

### Dictionary Loop Output

```
TASK [Display user information] **********************************************
ok: [localhost] => (item={'name': 'alice', 'uid': 1001, 'shell': '/bin/bash'}) => {
    "msg": "User alice has UID 1001 and shell /bin/bash"
}
```

### Loop with Label (Clean Output)

```
TASK [Loop with label (cleaner output)] **************************************
changed: [localhost] => (item=service1)
changed: [localhost] => (item=service2)
```

## Loop Syntax Reference

### Basic Loop
```yaml
loop:
  - item1
  - item2
```

### Loop with Variable
```yaml
loop: "{{ my_list }}"
```

### Loop with Range
```yaml
loop: "{{ range(1, 10) | list }}"  # 1 to 9
```

### Loop with Filters
```yaml
loop: "{{ users | selectattr('active') | list }}"
```

### Nested Loop (Product)
```yaml
loop: "{{ list1 | product(list2) | list }}"
```

## Troubleshooting

### Problem: Loop not iterating

**Solution:** Check variable type and content
```yaml
- name: Debug loop variable
  debug:
    var: my_list
```

### Problem: Need to access loop index

**Solution:** Use `loop_control` with `index_var`
```yaml
loop_control:
  index_var: idx
```

### Problem: Too much output in loops

**Solution:** Use `loop_control` with `label`
```yaml
loop_control:
  label: "{{ item.name }}"
```

## Additional Exercises

1. Create a loop that creates 10 numbered directories
2. Loop over a dictionary to create user accounts (simulated)
3. Use nested loops to create a matrix of files
4. Combine loops with conditionals to filter items
5. Create a loop with range() to generate sequential configs

## What You Learned

✅ Basic loop syntax with `loop` keyword  
✅ Looping over lists and variables  
✅ Looping over dictionaries (list of dicts)  
✅ Accessing dictionary values in loops  
✅ Loop control features (loop_var, index_var, label)  
✅ Nested loops using product filter  
✅ Combining loops with conditions  
✅ Pausing between loop iterations  

## Next Steps

Move on to [Exercise 09: File Management](../exercise-09/README.md) to learn advanced file and directory operations.
