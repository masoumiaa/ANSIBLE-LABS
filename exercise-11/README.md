# Exercise 11: Using Variables

## Learning Objectives

- Define and use variables
- Understand variable precedence
- Use different variable types
- Access variable values

## Prerequisites

- Completed Exercise 10
- Understanding of YAML data types

## Steps

### Step 1: Basic Variable Usage

Create a playbook with simple variables:

```bash
cat > basic-variables.yml << 'EOF'
---
- name: Basic Variable Usage
  hosts: localhost
  connection: local
  vars:
    app_name: "MyApplication"
    app_version: "1.0.0"
    app_port: 8080
    debug_enabled: true
  
  tasks:
    - name: Display application information
      debug:
        msg: |
          Application: {{ app_name }}
          Version: {{ app_version }}
          Port: {{ app_port }}
          Debug: {{ debug_enabled }}
    
    - name: Create configuration with variables
      copy:
        content: |
          # {{ app_name }} Configuration
          version={{ app_version }}
          port={{ app_port }}
          debug={{ debug_enabled | lower }}
        dest: /tmp/app-config.ini
    
    - name: Use variables in file paths
      file:
        path: "/tmp/{{ app_name }}"
        state: directory
    
    - name: Create file with variable name
      copy:
        content: "Application data for version {{ app_version }}\n"
        dest: "/tmp/{{ app_name }}/data.txt"
EOF
```

### Step 2: Run Basic Variables Playbook

```bash
ansible-playbook basic-variables.yml

# Check created files
cat /tmp/app-config.ini
```

### Step 3: Variable Types

Create a playbook demonstrating different variable types:

```bash
cat > variable-types.yml << 'EOF'
---
- name: Variable Types
  hosts: localhost
  connection: local
  vars:
    # String variables
    string_var: "Hello World"
    
    # Number variables
    integer_var: 42
    float_var: 3.14
    
    # Boolean variables
    bool_true: true
    bool_false: false
    
    # List variables
    fruits:
      - apple
      - banana
      - cherry
    
    # Dictionary variables
    server:
      hostname: web01
      ip: 192.168.1.10
      port: 80
    
    # Nested dictionary
    database:
      primary:
        host: db01
        port: 5432
      replica:
        host: db02
        port: 5432
  
  tasks:
    - name: Display string variable
      debug:
        msg: "String: {{ string_var }}"
    
    - name: Display number variables
      debug:
        msg: "Integer: {{ integer_var }}, Float: {{ float_var }}"
    
    - name: Display boolean variables
      debug:
        msg: "True: {{ bool_true }}, False: {{ bool_false }}"
    
    - name: Display list items
      debug:
        msg: "Fruit {{ item_index }}: {{ item }}"
      loop: "{{ fruits }}"
      loop_control:
        index_var: item_index
    
    - name: Access dictionary values
      debug:
        msg: "Server: {{ server.hostname }} at {{ server.ip }}:{{ server.port }}"
    
    - name: Access nested dictionary
      debug:
        msg: "Primary DB: {{ database.primary.host }}:{{ database.primary.port }}"
    
    - name: Use bracket notation
      debug:
        msg: "Server IP: {{ server['ip'] }}"
    
    - name: Create config from variables
      copy:
        content: |
          # Server Configuration
          [server]
          hostname={{ server.hostname }}
          ip={{ server.ip }}
          port={{ server.port }}
          
          [database]
          primary={{ database.primary.host }}:{{ database.primary.port }}
          replica={{ database.replica.host }}:{{ database.replica.port }}
          
          [fruits]
          {% for fruit in fruits %}
          item={{ fruit }}
          {% endfor %}
        dest: /tmp/config-from-vars.ini
EOF
```

### Step 4: Test Variable Types

```bash
ansible-playbook variable-types.yml
cat /tmp/config-from-vars.ini
```

### Step 5: Command-Line Variables

Create a playbook that accepts command-line variables:

```bash
cat > cli-variables.yml << 'EOF'
---
- name: Command-Line Variables
  hosts: localhost
  connection: local
  vars:
    default_env: "development"
    default_port: 8080
  
  tasks:
    - name: Display environment (with default)
      debug:
        msg: "Environment: {{ env | default(default_env) }}"
    
    - name: Display port (with default)
      debug:
        msg: "Port: {{ port | default(default_port) }}"
    
    - name: Display custom message (if provided)
      debug:
        msg: "Custom message: {{ custom_message }}"
      when: custom_message is defined
    
    - name: Create environment-specific file
      copy:
        content: |
          Environment: {{ env | default(default_env) }}
          Port: {{ port | default(default_port) }}
          Timestamp: {{ ansible_date_time.iso8601 }}
        dest: "/tmp/env-{{ env | default(default_env) }}.conf"
EOF
```

### Step 6: Test Command-Line Variables

```bash
# Run with defaults
ansible-playbook cli-variables.yml

# Run with custom variables
ansible-playbook cli-variables.yml -e "env=production"
ansible-playbook cli-variables.yml -e "env=staging port=9090"
ansible-playbook cli-variables.yml -e "env=test custom_message='Hello from CLI'"
ansible-playbook cli-variables.yml -e '{"env":"production","port":8443}'
```

### Step 7: Register Variables

Create a playbook using register to create variables:

```bash
cat > register-variables.yml << 'EOF'
---
- name: Register Variables
  hosts: localhost
  connection: local
  
  tasks:
    - name: Get current date
      command: date +%Y-%m-%d
      register: current_date
      changed_when: false
    
    - name: Get system hostname
      command: hostname
      register: system_hostname
      changed_when: false
    
    - name: Create directory
      file:
        path: /tmp/registered-vars
        state: directory
      register: dir_creation
    
    - name: Display registered variables
      debug:
        msg: |
          Date: {{ current_date.stdout }}
          Hostname: {{ system_hostname.stdout }}
          Directory created: {{ dir_creation.changed }}
    
    - name: Use registered variable in filename
      copy:
        content: |
          Report Date: {{ current_date.stdout }}
          System: {{ system_hostname.stdout }}
        dest: "/tmp/registered-vars/report-{{ current_date.stdout }}.txt"
    
    - name: Read file back
      slurp:
        src: "/tmp/registered-vars/report-{{ current_date.stdout }}.txt"
      register: file_content
    
    - name: Display file content
      debug:
        msg: "{{ file_content.content | b64decode }}"
    
    - name: Complex register usage
      shell: |
        echo "Line 1"
        echo "Line 2"
        echo "Line 3"
      register: multi_line
      changed_when: false
    
    - name: Display multiline output
      debug:
        var: multi_line.stdout_lines
EOF
```

### Step 8: Test Register Variables

```bash
ansible-playbook register-variables.yml
```

### Step 9: Facts as Variables

Create a playbook using ansible facts:

```bash
cat > facts-as-variables.yml << 'EOF'
---
- name: Facts as Variables
  hosts: localhost
  connection: local
  gather_facts: yes
  
  tasks:
    - name: Display system facts
      debug:
        msg: |
          Hostname: {{ ansible_hostname }}
          OS: {{ ansible_distribution }} {{ ansible_distribution_version }}
          Architecture: {{ ansible_architecture }}
          Python: {{ ansible_python_version }}
    
    - name: Display network facts
      debug:
        msg: "IP Address: {{ ansible_default_ipv4.address | default('N/A') }}"
    
    - name: Display memory facts
      debug:
        msg: |
          Total Memory: {{ ansible_memtotal_mb }} MB
          Free Memory: {{ ansible_memfree_mb }} MB
    
    - name: Create system info file using facts
      copy:
        content: |
          System Information Report
          =========================
          Hostname: {{ ansible_hostname }}
          FQDN: {{ ansible_fqdn }}
          
          Operating System:
          - Distribution: {{ ansible_distribution }}
          - Version: {{ ansible_distribution_version }}
          - OS Family: {{ ansible_os_family }}
          
          Hardware:
          - Architecture: {{ ansible_architecture }}
          - Processor: {{ ansible_processor[0] | default('Unknown') }}
          - CPU Cores: {{ ansible_processor_cores | default('N/A') }}
          - Memory: {{ ansible_memtotal_mb }} MB
          
          Network:
          - IP: {{ ansible_default_ipv4.address | default('N/A') }}
          - Gateway: {{ ansible_default_ipv4.gateway | default('N/A') }}
          
          Python: {{ ansible_python_version }}
          
          Generated: {{ ansible_date_time.iso8601 }}
        dest: /tmp/system-info.txt
    
    - name: Display custom facts (if any)
      debug:
        var: ansible_local
      when: ansible_local is defined
EOF
```

### Step 10: Test Facts as Variables

```bash
ansible-playbook facts-as-variables.yml
cat /tmp/system-info.txt
```

## Expected Results

### Variable Display Output

```
TASK [Display application information] ***************************************
ok: [localhost] => {
    "msg": "Application: MyApplication\nVersion: 1.0.0\nPort: 8080\nDebug: True\n"
}
```

### Dictionary Access

```
TASK [Access dictionary values] **********************************************
ok: [localhost] => {
    "msg": "Server: web01 at 192.168.1.10:80"
}
```

### Register Variable

```json
{
    "changed": false,
    "cmd": "date +%Y-%m-%d",
    "stdout": "2026-01-28",
    "rc": 0
}
```

## Variable Precedence (Lowest to Highest)

1. Role defaults
2. Inventory variables
3. Playbook vars
4. Playbook vars_files
5. Host facts
6. Registered variables
7. Set_facts
8. Extra vars (-e from command line) **[HIGHEST]**

## Variable Access Methods

### Dot Notation
```yaml
{{ server.hostname }}
{{ database.primary.host }}
```

### Bracket Notation
```yaml
{{ server['hostname'] }}
{{ database['primary']['host'] }}
```

### With Filters
```yaml
{{ string_var | upper }}
{{ list_var | length }}
{{ dict_var | to_json }}
```

## Troubleshooting

### Problem: Variable not found

**Solution:** Check variable spelling and scope
```yaml
- name: Debug variable
  debug:
    var: my_variable
```

### Problem: Variable is undefined

**Solution:** Use default filter
```yaml
{{ my_var | default('default_value') }}
```

### Problem: Cannot access nested value

**Solution:** Use bracket notation or check structure
```yaml
- name: Show full structure
  debug:
    var: my_dict
```

## Additional Exercises

1. Create a playbook with 10 different variables of mixed types
2. Use command-line variables to control playbook behavior
3. Build a system inventory report using facts
4. Create nested dictionaries for complex configurations
5. Practice variable precedence with multiple sources

## What You Learned

✅ Defining variables in playbooks  
✅ Different variable types (strings, numbers, booleans, lists, dictionaries)  
✅ Accessing variable values with dot and bracket notation  
✅ Using command-line variables with -e  
✅ Registering task output as variables  
✅ Using Ansible facts as variables  
✅ Variable precedence rules  
✅ Default values for undefined variables  

## Next Steps

Move on to [Exercise 12: Variable Files](../exercise-12/README.md) to learn about externalizing variables.
