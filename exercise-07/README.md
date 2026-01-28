# Exercise 07: Conditional Execution

## Learning Objectives

- Use `when` statements for conditional execution
- Compare variables and facts
- Use logical operators
- Implement complex conditions

## Prerequisites

- Completed Exercise 06
- Understanding of boolean logic

## Steps

### Step 1: Basic When Statement

Create a playbook with simple conditions:

```bash
cat > basic-when.yml << 'EOF'
---
- name: Basic Conditional Execution
  hosts: localhost
  connection: local
  vars:
    deploy_env: production
    debug_mode: false
  
  tasks:
    - name: Task that always runs
      debug:
        msg: "This task always runs"
    
    - name: Task for production only
      debug:
        msg: "Running in production environment"
      when: deploy_env == "production"
    
    - name: Task for development only
      debug:
        msg: "Running in development environment"
      when: deploy_env == "development"
    
    - name: Task when debug is enabled
      debug:
        msg: "Debug mode is ON"
      when: debug_mode == true
    
    - name: Task when debug is disabled
      debug:
        msg: "Debug mode is OFF"
      when: debug_mode == false
EOF
```

### Step 2: Run Basic Conditional Playbook

```bash
# Run with default variables
ansible-playbook basic-when.yml

# Run with different variables
ansible-playbook basic-when.yml -e "deploy_env=development"
ansible-playbook basic-when.yml -e "debug_mode=true"
```

### Step 3: Conditions Based on Facts

Create a playbook using system facts:

```bash
cat > fact-conditions.yml << 'EOF'
---
- name: Conditional Execution Based on Facts
  hosts: localhost
  connection: local
  gather_facts: yes
  
  tasks:
    - name: Display OS family
      debug:
        msg: "OS Family: {{ ansible_os_family }}"
    
    - name: Task for Debian-based systems
      debug:
        msg: "This is a Debian/Ubuntu system"
      when: ansible_os_family == "Debian"
    
    - name: Task for RedHat-based systems
      debug:
        msg: "This is a RedHat/CentOS system"
      when: ansible_os_family == "RedHat"
    
    - name: Task for systems with enough memory
      debug:
        msg: "System has sufficient memory (>2GB)"
      when: ansible_memtotal_mb > 2048
    
    - name: Task for systems with low memory
      debug:
        msg: "Warning: Low memory system (<2GB)"
      when: ansible_memtotal_mb < 2048
    
    - name: Task for specific Python version
      debug:
        msg: "Python 3 is installed"
      when: ansible_python_version is version('3.0', '>=')
EOF
```

### Step 4: Test Fact Conditions

```bash
ansible-playbook fact-conditions.yml
```

### Step 5: Logical Operators

Create a playbook with complex conditions:

```bash
cat > logical-operators.yml << 'EOF'
---
- name: Logical Operators in Conditions
  hosts: localhost
  connection: local
  vars:
    app_env: production
    enable_backup: true
    enable_monitoring: true
    server_role: webserver
  
  tasks:
    - name: AND operator - Both conditions must be true
      debug:
        msg: "Production with backup enabled"
      when: app_env == "production" and enable_backup == true
    
    - name: OR operator - Either condition can be true
      debug:
        msg: "Either monitoring or backup is enabled"
      when: enable_backup == true or enable_monitoring == true
    
    - name: NOT operator - Condition must be false
      debug:
        msg: "Not in production environment"
      when: app_env != "production"
    
    - name: Multiple AND conditions
      debug:
        msg: "Production webserver with monitoring"
      when:
        - app_env == "production"
        - server_role == "webserver"
        - enable_monitoring == true
    
    - name: Parentheses for complex logic
      debug:
        msg: "Complex condition met"
      when: (app_env == "production" and enable_backup) or server_role == "database"
    
    - name: IN operator - Check if value in list
      debug:
        msg: "Environment is in allowed list"
      when: app_env in ["production", "staging", "qa"]
EOF
```

### Step 6: Test Logical Operators

```bash
# Test with default values
ansible-playbook logical-operators.yml

# Test with different values
ansible-playbook logical-operators.yml -e "app_env=development"
ansible-playbook logical-operators.yml -e "server_role=database"
```

### Step 7: Checking Variable Existence

Create a playbook that checks if variables are defined:

```bash
cat > variable-checks.yml << 'EOF'
---
- name: Variable Existence Checks
  hosts: localhost
  connection: local
  vars:
    defined_var: "I exist"
    empty_var: ""
  
  tasks:
    - name: Check if variable is defined
      debug:
        msg: "Variable 'defined_var' is defined"
      when: defined_var is defined
    
    - name: Check if variable is undefined
      debug:
        msg: "Variable 'undefined_var' is not defined"
      when: undefined_var is not defined
    
    - name: Check if variable is not empty
      debug:
        msg: "Variable has content"
      when: defined_var | length > 0
    
    - name: Check if variable is empty
      debug:
        msg: "Variable is empty string"
      when: empty_var | length == 0
    
    - name: Use default value for undefined variable
      debug:
        msg: "Value: {{ undefined_var | default('default_value') }}"
    
    - name: Skip if variable is undefined
      debug:
        msg: "This only runs if my_var is defined"
      when: my_var is defined and my_var != ""
EOF
```

### Step 8: Test Variable Checks

```bash
# Run without extra variables
ansible-playbook variable-checks.yml

# Run with extra variable
ansible-playbook variable-checks.yml -e "my_var='Hello World'"
```

### Step 9: Register and Conditional

Create a playbook using task results in conditions:

```bash
cat > register-conditions.yml << 'EOF'
---
- name: Conditional Based on Task Results
  hosts: localhost
  connection: local
  
  tasks:
    - name: Check if directory exists
      stat:
        path: /tmp/test-dir
      register: dir_check
    
    - name: Create directory if it doesn't exist
      file:
        path: /tmp/test-dir
        state: directory
      when: not dir_check.stat.exists
    
    - name: Message if directory already existed
      debug:
        msg: "Directory already exists"
      when: dir_check.stat.exists
    
    - name: Run a command
      command: echo "Hello Ansible"
      register: command_result
      changed_when: false
    
    - name: Display output if command succeeded
      debug:
        msg: "Command output: {{ command_result.stdout }}"
      when: command_result.rc == 0
    
    - name: Try a command that might fail
      command: /bin/false
      register: failed_command
      ignore_errors: yes
    
    - name: Handle failure
      debug:
        msg: "Previous command failed as expected"
      when: failed_command.failed
    
    - name: Check file content
      command: cat /tmp/test-dir/test.txt
      register: file_content
      ignore_errors: yes
    
    - name: Create file if it doesn't exist
      copy:
        content: "Test file content\n"
        dest: /tmp/test-dir/test.txt
      when: file_content.rc != 0
EOF
```

### Step 10: Test Register Conditions

```bash
# First run - directory doesn't exist
ansible-playbook register-conditions.yml

# Second run - directory exists
ansible-playbook register-conditions.yml
```

## Expected Results

### Conditional Task Skipped

```
TASK [Task for development only] *********************************************
skipping: [localhost]
```

### Conditional Task Executed

```
TASK [Task for production only] **********************************************
ok: [localhost] => {
    "msg": "Running in production environment"
}
```

### Multiple Conditions

```
TASK [Multiple AND conditions] ***********************************************
ok: [localhost] => {
    "msg": "Production webserver with monitoring"
}
```

## Common Condition Patterns

### Equality Checks
```yaml
when: variable == "value"
when: variable != "value"
```

### Numeric Comparisons
```yaml
when: count > 10
when: count >= 10
when: count < 10
when: count <= 10
```

### Boolean Checks
```yaml
when: boolean_var
when: not boolean_var
when: boolean_var == true
```

### Variable Tests
```yaml
when: variable is defined
when: variable is not defined
when: variable is none
when: result.failed
when: result.changed
```

### String Operations
```yaml
when: string_var in ["value1", "value2"]
when: "'substring' in string_var"
when: string_var | length > 0
```

## Troubleshooting

### Problem: Condition not working as expected

**Solution:** Use debug to check variable values
```yaml
- name: Debug variable
  debug:
    var: my_variable
```

### Problem: String vs Boolean comparison

**Solution:** Ensure correct type
```yaml
when: string_var == "true"  # String comparison
when: bool_var == true       # Boolean comparison
```

### Problem: Undefined variable in condition

**Solution:** Use `| default()` filter
```yaml
when: (my_var | default('')) != ''
```

## Additional Exercises

1. Create conditions based on multiple system facts
2. Implement role-based task execution using variables
3. Use register with conditions to create task dependencies
4. Build a playbook with nested conditions
5. Create environment-specific deployments using when statements

## What You Learned

✅ Using `when` statements for conditional execution  
✅ Conditions based on variables and facts  
✅ Logical operators (and, or, not)  
✅ Checking variable existence and values  
✅ Using task results (register) in conditions  
✅ Complex conditional logic  
✅ String and numeric comparisons  
✅ Boolean evaluations  

## Next Steps

Move on to [Exercise 08: Loops in Playbooks](../exercise-08/README.md) to learn how to iterate over data structures.
