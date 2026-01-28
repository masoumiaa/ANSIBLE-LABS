# Exercise 05: Multiple Tasks

## Learning Objectives

- Chain multiple tasks together
- Use task output in subsequent tasks
- Understand task dependencies
- Manage task execution flow

## Prerequisites

- Completed Exercise 04
- Understanding of playbook structure

## Steps

### Step 1: Create a Playbook with Task Dependencies

Create a playbook where tasks depend on each other:

```bash
cat > dependent-tasks.yml << 'EOF'
---
- name: Tasks with Dependencies
  hosts: localhost
  connection: local
  
  tasks:
    - name: Create parent directory
      file:
        path: /tmp/ansible-tasks
        state: directory
        mode: '0755'
    
    - name: Create subdirectory
      file:
        path: /tmp/ansible-tasks/data
        state: directory
        mode: '0755'
    
    - name: Create configuration file
      copy:
        content: |
          # Configuration File
          app_name: My Application
          version: 1.0
          environment: development
        dest: /tmp/ansible-tasks/config.conf
    
    - name: Create data file
      copy:
        content: |
          Sample data line 1
          Sample data line 2
          Sample data line 3
        dest: /tmp/ansible-tasks/data/sample.txt
    
    - name: List all created files
      command: find /tmp/ansible-tasks -type f
      register: created_files
    
    - name: Display created files
      debug:
        msg: "Created files: {{ created_files.stdout_lines }}"
EOF
```

### Step 2: Run and Verify

```bash
# Execute the playbook
ansible-playbook dependent-tasks.yml

# Verify the directory structure
tree /tmp/ansible-tasks  # or ls -R /tmp/ansible-tasks
```

### Step 3: Using Register and Variables

Create a playbook that captures and uses task output:

```bash
cat > register-example.yml << 'EOF'
---
- name: Using Register with Multiple Tasks
  hosts: localhost
  connection: local
  
  tasks:
    - name: Get current date
      command: date +%Y-%m-%d
      register: current_date
    
    - name: Get current time
      command: date +%H:%M:%S
      register: current_time
    
    - name: Create timestamp file
      copy:
        content: |
          Timestamp Information
          =====================
          Date: {{ current_date.stdout }}
          Time: {{ current_time.stdout }}
          Full output: {{ current_date.stdout }} {{ current_time.stdout }}
        dest: /tmp/timestamp.txt
    
    - name: Read the created file
      slurp:
        src: /tmp/timestamp.txt
      register: file_content
    
    - name: Display file content
      debug:
        msg: "{{ file_content.content | b64decode }}"
    
    - name: Check if file exists and get stats
      stat:
        path: /tmp/timestamp.txt
      register: file_stats
    
    - name: Display file information
      debug:
        msg: |
          File exists: {{ file_stats.stat.exists }}
          File size: {{ file_stats.stat.size }} bytes
          File mode: {{ file_stats.stat.mode }}
EOF
```

### Step 4: Execute Register Example

```bash
ansible-playbook register-example.yml
```

### Step 5: Sequential Task Processing

Create a playbook that processes data sequentially:

```bash
cat > sequential-tasks.yml << 'EOF'
---
- name: Sequential Data Processing
  hosts: localhost
  connection: local
  
  tasks:
    - name: Step 1 - Create workspace
      file:
        path: /tmp/processing
        state: directory
    
    - name: Step 2 - Create input data
      copy:
        content: |
          apple
          banana
          cherry
          date
          elderberry
        dest: /tmp/processing/input.txt
    
    - name: Step 3 - Count lines in input
      shell: wc -l /tmp/processing/input.txt | awk '{print $1}'
      register: line_count
    
    - name: Step 4 - Display count
      debug:
        msg: "Input file contains {{ line_count.stdout }} lines"
    
    - name: Step 5 - Sort the data
      shell: sort /tmp/processing/input.txt > /tmp/processing/sorted.txt
    
    - name: Step 6 - Create uppercase version
      shell: cat /tmp/processing/sorted.txt | tr '[:lower:]' '[:upper:]' > /tmp/processing/uppercase.txt
    
    - name: Step 7 - Read all versions
      shell: |
        echo "=== Original ==="
        cat /tmp/processing/input.txt
        echo "=== Sorted ==="
        cat /tmp/processing/sorted.txt
        echo "=== Uppercase ==="
        cat /tmp/processing/uppercase.txt
      register: all_versions
    
    - name: Step 8 - Display all versions
      debug:
        msg: "{{ all_versions.stdout_lines }}"
    
    - name: Step 9 - Create summary
      copy:
        content: |
          Processing Summary
          ==================
          Total items processed: {{ line_count.stdout }}
          Files created:
            - input.txt
            - sorted.txt
            - uppercase.txt
          
          Processing completed successfully!
        dest: /tmp/processing/summary.txt
    
    - name: Step 10 - Final verification
      command: ls -lh /tmp/processing/
      register: final_list
    
    - name: Display final directory listing
      debug:
        var: final_list.stdout_lines
EOF
```

### Step 6: Run Sequential Processing

```bash
ansible-playbook sequential-tasks.yml
```

### Step 7: Task Failure Handling

Create a playbook with error handling across tasks:

```bash
cat > error-handling.yml << 'EOF'
---
- name: Task Execution with Error Handling
  hosts: localhost
  connection: local
  
  tasks:
    - name: Task 1 - Always succeeds
      debug:
        msg: "Task 1 completed successfully"
    
    - name: Task 2 - Might fail (ignored)
      command: /bin/false
      register: task2_result
      ignore_errors: yes
    
    - name: Task 3 - Check previous task status
      debug:
        msg: "Task 2 failed: {{ task2_result.failed }}, Return code: {{ task2_result.rc }}"
    
    - name: Task 4 - Conditional execution
      debug:
        msg: "This runs because we handled the error"
    
    - name: Task 5 - Create success marker
      copy:
        content: "All tasks completed (with error handling)\n"
        dest: /tmp/task-completion.txt
    
    - name: Task 6 - Verify completion
      stat:
        path: /tmp/task-completion.txt
      register: completion_check
    
    - name: Task 7 - Final status
      debug:
        msg: "Playbook completed. Marker file exists: {{ completion_check.stat.exists }}"
EOF
```

### Step 8: Test Error Handling

```bash
ansible-playbook error-handling.yml
```

## Expected Results

### Task Chain Output

```
TASK [Create parent directory] ***********************************************
changed: [localhost]

TASK [Create subdirectory] ***************************************************
changed: [localhost]

TASK [Create configuration file] *********************************************
changed: [localhost]

TASK [Display created files] *************************************************
ok: [localhost] => {
    "msg": "Created files: ['/tmp/ansible-tasks/config.conf', '/tmp/ansible-tasks/data/sample.txt']"
}
```

### Register Output

```json
{
    "changed": true,
    "cmd": "date +%Y-%m-%d",
    "stdout": "2026-01-28",
    "rc": 0
}
```

## Key Concepts

### Task Execution Order
- Tasks execute sequentially from top to bottom
- Each task must succeed before the next runs (unless errors ignored)
- Use `register` to capture output for later tasks

### Register Variable Contents
```yaml
register_variable:
  changed: true/false
  failed: true/false
  rc: return_code
  stdout: command_output
  stderr: error_output
  stdout_lines: ["line1", "line2"]
```

## Troubleshooting

### Problem: Task fails and stops playbook

**Solution:** Use `ignore_errors: yes`
```yaml
- name: Task that might fail
  command: /some/command
  ignore_errors: yes
```

### Problem: Need to access previous task output

**Solution:** Use `register` and reference with `{{ var_name.stdout }}`

### Problem: Task depends on previous task success

**Solution:** Use `when` condition
```yaml
- name: Second task
  command: echo "success"
  when: first_task.rc == 0
```

## Additional Exercises

1. Create a 10-task playbook that builds a complete directory structure
2. Write a playbook that processes a file through multiple transformations
3. Create tasks that depend on previous task outputs
4. Implement error handling for a chain of tasks
5. Build a playbook that validates each step before proceeding

## What You Learned

✅ Chaining multiple tasks together  
✅ Using register to capture task output  
✅ Referencing previous task results  
✅ Sequential data processing  
✅ Error handling in task chains  
✅ Conditional task execution  
✅ Using stat module for file verification  

## Next Steps

Move on to [Exercise 06: Working with Handlers](../exercise-06/README.md) to learn about event-driven task execution.
