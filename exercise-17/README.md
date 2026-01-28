# Exercise 17: Error Handling

## Learning Objectives

- Handle task failures gracefully
- Use blocks for error handling
- Implement rescue and always sections
- Control error behavior

## Prerequisites

- Completed Exercise 16
- Understanding of task execution

## Steps

### Step 1: Basic Error Handling

Create a playbook with basic error handling:

```bash
cat > basic-errors.yml << 'EOF'
---
- name: Basic Error Handling
  hosts: localhost
  connection: local
  
  tasks:
    - name: Task that always succeeds
      debug:
        msg: "This task succeeds"
    
    - name: Task that fails (but continues)
      command: /bin/false
      ignore_errors: yes
    
    - name: This task still runs
      debug:
        msg: "Execution continues despite previous failure"
    
    - name: Check command result
      command: /bin/true
      register: result
    
    - name: Display result
      debug:
        msg: "Return code: {{ result.rc }}"
    
    - name: Task with explicit failure
      fail:
        msg: "This task fails intentionally"
      when: false  # Set to true to see failure
    
    - name: Final task
      debug:
        msg: "Playbook completed successfully"
EOF
```

### Step 2: Run Basic Error Handling

```bash
ansible-playbook basic-errors.yml
```

### Step 3: Using Blocks for Error Handling

Create a playbook using blocks:

```bash
cat > blocks-rescue.yml << 'EOF'
---
- name: Blocks with Rescue and Always
  hosts: localhost
  connection: local
  
  tasks:
    - name: Block with error handling
      block:
        - name: Attempt risky operation
          debug:
            msg: "Attempting operation..."
        
        - name: Create a file
          copy:
            content: "Test content\n"
            dest: /tmp/test-block.txt
        
        - name: Task that might fail
          command: /bin/false
          changed_when: false
        
        - name: This won't run if previous task fails
          debug:
            msg: "This message won't appear"
      
      rescue:
        - name: Handle the error
          debug:
            msg: "An error occurred, but we're handling it!"
        
        - name: Log the error
          copy:
            content: "Error occurred at {{ ansible_date_time.iso8601 }}\n"
            dest: /tmp/error-log.txt
        
        - name: Attempt recovery
          debug:
            msg: "Attempting to recover from error"
      
      always:
        - name: Cleanup (always runs)
          debug:
            msg: "Cleanup: This always runs regardless of success or failure"
        
        - name: Create status file
          copy:
            content: "Block executed at {{ ansible_date_time.iso8601 }}\n"
            dest: /tmp/block-status.txt
EOF
```

### Step 4: Test Block Error Handling

```bash
ansible-playbook blocks-rescue.yml
cat /tmp/error-log.txt
cat /tmp/block-status.txt
```

### Step 5: Multiple Error Scenarios

Create a playbook with various error scenarios:

```bash
cat > error-scenarios.yml << 'EOF'
---
- name: Multiple Error Scenarios
  hosts: localhost
  connection: local
  
  tasks:
    - name: Scenario 1 - Ignore errors
      block:
        - name: Command that fails
          command: /bin/false
          ignore_errors: yes
        
        - name: Continue after ignored error
          debug:
            msg: "Continuing execution after ignored error"
    
    - name: Scenario 2 - Failed when condition
      block:
        - name: Check disk space (simulated)
          shell: echo "50"
          register: disk_usage
          changed_when: false
        
        - name: Fail if disk usage too high
          fail:
            msg: "Disk usage is {{ disk_usage.stdout }}% - too high!"
          when: disk_usage.stdout | int > 80
        
        - name: Success message
          debug:
            msg: "Disk usage is acceptable: {{ disk_usage.stdout }}%"
    
    - name: Scenario 3 - Changed when condition
      block:
        - name: Check if file exists
          stat:
            path: /tmp/marker.txt
          register: file_check
        
        - name: Create file only if missing
          copy:
            content: "Created\n"
            dest: /tmp/marker.txt
          changed_when: not file_check.stat.exists
        
        - name: Report status
          debug:
            msg: "File {{ 'created' if not file_check.stat.exists else 'already exists' }}"
    
    - name: Scenario 4 - Any errors fatal
      block:
        - name: Critical operation
          debug:
            msg: "Performing critical operation"
        
        - name: Must succeed
          command: /bin/true
          changed_when: false
      any_errors_fatal: yes
EOF
```

### Step 6: Test Error Scenarios

```bash
ansible-playbook error-scenarios.yml
```

### Step 7: Advanced Recovery Patterns

Create a playbook with advanced recovery:

```bash
cat > advanced-recovery.yml << 'EOF'
---
- name: Advanced Recovery Patterns
  hosts: localhost
  connection: local
  vars:
    max_retries: 3
    retry_delay: 2
  
  tasks:
    - name: Retry pattern with block
      block:
        - name: Operation that might fail
          shell: |
            # Simulate intermittent failure
            if [ $((RANDOM % 3)) -eq 0 ]; then
              echo "Success"
              exit 0
            else
              echo "Failed"
              exit 1
            fi
          register: operation_result
          until: operation_result.rc == 0
          retries: "{{ max_retries }}"
          delay: "{{ retry_delay }}"
          changed_when: false
      
      rescue:
        - name: All retries failed
          debug:
            msg: "Operation failed after {{ max_retries }} retries"
    
    - name: Fallback strategy
      block:
        - name: Try primary method
          command: /opt/primary-tool --execute
          register: primary_result
          ignore_errors: yes
        
        - name: Use fallback if primary fails
          command: /usr/bin/alternative-tool --execute
          when: primary_result.failed
          register: fallback_result
          ignore_errors: yes
        
        - name: Check if any method succeeded
          assert:
            that:
              - primary_result.rc == 0 or fallback_result.rc == 0
            fail_msg: "Both primary and fallback methods failed"
            success_msg: "Operation completed successfully"
      
      rescue:
        - name: Log complete failure
          copy:
            content: |
              Complete Failure Log
              ====================
              Timestamp: {{ ansible_date_time.iso8601 }}
              Primary result: {{ primary_result.rc | default('not run') }}
              Fallback result: {{ fallback_result.rc | default('not run') }}
            dest: /tmp/failure-log.txt
      
      always:
        - name: Send notification (simulated)
          debug:
            msg: "Notification sent about operation status"
EOF
```

### Step 8: Test Advanced Recovery

```bash
ansible-playbook advanced-recovery.yml --syntax-check
# Note: This playbook has commands that don't exist, demonstrating error handling
```

### Step 9: Validation and Assertions

Create a playbook with validation:

```bash
cat > validation-assertions.yml << 'EOF'
---
- name: Validation and Assertions
  hosts: localhost
  connection: local
  gather_facts: yes
  vars:
    min_memory_mb: 1024
    required_packages:
      - python3
    app_port: 8080
  
  tasks:
    - name: Validate system requirements
      block:
        - name: Check memory
          assert:
            that:
              - ansible_memtotal_mb >= min_memory_mb
            fail_msg: "Insufficient memory: {{ ansible_memtotal_mb }}MB (minimum {{ min_memory_mb }}MB required)"
            success_msg: "Memory check passed: {{ ansible_memtotal_mb }}MB available"
        
        - name: Check disk space
          shell: df -m /tmp | tail -1 | awk '{print $4}'
          register: disk_space
          changed_when: false
        
        - name: Validate disk space
          assert:
            that:
              - disk_space.stdout | int > 100
            fail_msg: "Insufficient disk space: {{ disk_space.stdout }}MB"
            success_msg: "Disk space check passed"
        
        - name: Check port availability
          wait_for:
            port: "{{ app_port }}"
            state: stopped
            timeout: 1
          ignore_errors: yes
          register: port_check
        
        - name: Validate port is free
          assert:
            that:
              - port_check.failed
            fail_msg: "Port {{ app_port }} is already in use"
            success_msg: "Port {{ app_port }} is available"
      
      rescue:
        - name: Validation failed
          debug:
            msg: "System validation failed - cannot proceed with deployment"
        
        - name: Create validation report
          copy:
            content: |
              Validation Failed
              =================
              Date: {{ ansible_date_time.iso8601 }}
              Host: {{ ansible_hostname }}
              
              Checks performed:
              - Memory: {{ ansible_memtotal_mb }}MB (required: {{ min_memory_mb }}MB)
              - Disk: {{ disk_space.stdout | default('unknown') }}MB
              - Port {{ app_port }}: {{ 'in use' if not port_check.failed else 'available' }}
              
              Recommendation: Fix validation errors before deploying
            dest: /tmp/validation-failed.txt
        
        - name: Abort playbook
          fail:
            msg: "Aborting due to validation failure"
EOF
```

### Step 10: Test Validation

```bash
ansible-playbook validation-assertions.yml
```

### Step 11: Complete Error Handling Example

Create a comprehensive error handling playbook:

```bash
cat > complete-error-handling.yml << 'EOF'
---
- name: Complete Error Handling Example
  hosts: localhost
  connection: local
  vars:
    deployment_id: "{{ 99999 | random | to_uuid }}"
    
  tasks:
    - name: Initialize deployment
      block:
        - name: Create deployment directory
          file:
            path: /tmp/deployment-{{ deployment_id }}
            state: directory
        
        - name: Log deployment start
          copy:
            content: |
              Deployment {{ deployment_id }}
              Started: {{ ansible_date_time.iso8601 }}
              Status: In Progress
            dest: /tmp/deployment-{{ deployment_id }}/status.log
      
      rescue:
        - name: Initialization failed
          fail:
            msg: "Cannot initialize deployment - critical error"
    
    - name: Deploy application
      block:
        - name: Download application (simulated)
          debug:
            msg: "Downloading application..."
        
        - name: Extract files (simulated)
          debug:
            msg: "Extracting files..."
        
        - name: Configure application (might fail)
          command: /bin/true  # Change to /bin/false to test failure
          register: config_result
          changed_when: false
        
        - name: Verify configuration
          assert:
            that:
              - config_result.rc == 0
            fail_msg: "Configuration verification failed"
        
        - name: Start application (simulated)
          debug:
            msg: "Starting application..."
      
      rescue:
        - name: Deployment failed - rollback
          debug:
            msg: "Deployment failed, initiating rollback..."
        
        - name: Stop application
          debug:
            msg: "Stopping application..."
        
        - name: Remove deployment files
          file:
            path: /tmp/deployment-{{ deployment_id }}
            state: absent
        
        - name: Log rollback
          copy:
            content: |
              Deployment {{ deployment_id }}
              Status: ROLLED BACK
              Timestamp: {{ ansible_date_time.iso8601 }}
              Reason: Configuration failed
            dest: /tmp/rollback-{{ deployment_id }}.log
        
        - name: Notify failure
          debug:
            msg: "Deployment failed and was rolled back successfully"
      
      always:
        - name: Cleanup temporary files
          debug:
            msg: "Cleaning up temporary files..."
        
        - name: Update deployment status
          copy:
            content: |
              Deployment {{ deployment_id }}
              Completed: {{ ansible_date_time.iso8601 }}
              Final Status: {{ 'SUCCESS' if config_result.rc == 0 else 'FAILED' }}
            dest: /tmp/deployment-{{ deployment_id }}/final-status.log
    
    - name: Post-deployment tasks
      block:
        - name: Run health checks
          debug:
            msg: "Running health checks..."
        
        - name: Send notification
          debug:
            msg: "Deployment completed successfully"
      
      when: config_result.rc == 0
EOF
```

### Step 12: Test Complete Error Handling

```bash
# Test successful deployment
ansible-playbook complete-error-handling.yml

# View logs
ls -la /tmp/deployment-*/
cat /tmp/deployment-*/status.log
cat /tmp/deployment-*/final-status.log
```

## Expected Results

### Block Rescue Output

```
TASK [Attempt risky operation] ***********************************************
ok: [localhost]

TASK [Task that might fail] **************************************************
fatal: [localhost]: FAILED! => ...

TASK [Handle the error] ******************************************************
ok: [localhost] => {
    "msg": "An error occurred, but we're handling it!"
}

TASK [Cleanup (always runs)] *************************************************
ok: [localhost]
```

### Assertion Failure

```
TASK [Validate disk space] ***************************************************
fatal: [localhost]: FAILED! => {
    "assertion": "disk_space.stdout | int > 100",
    "msg": "Insufficient disk space: 50MB"
}
```

## Error Handling Constructs

### Ignore Errors
```yaml
- name: Task
  command: /might/fail
  ignore_errors: yes
```

### Block/Rescue/Always
```yaml
- block:
    - name: Try
  rescue:
    - name: On error
  always:
    - name: Cleanup
```

### Failed When
```yaml
- name: Task
  command: /command
  register: result
  failed_when: result.rc != 0 or 'error' in result.stdout
```

### Changed When
```yaml
- name: Task
  command: /command
  changed_when: false
```

### Assertions
```yaml
- assert:
    that:
      - condition1
      - condition2
    fail_msg: "Failed"
    success_msg: "Passed"
```

## Troubleshooting

### Problem: Rescue block not executing

**Solution:** Ensure task actually fails
```yaml
- debug:
    var: task_result
```

### Problem: Always block skipped

**Solution:** Always blocks only run if block started

### Problem: Assertion not failing

**Solution:** Check condition syntax
```yaml
- debug:
    msg: "{{ condition }}"
```

## Additional Exercises

1. Create a deployment with automatic rollback on failure
2. Implement retry logic with exponential backoff
3. Build validation checks for system requirements
4. Create error recovery workflows
5. Design a health check system with automated recovery

## What You Learned

✅ Ignoring errors with ignore_errors  
✅ Using blocks for grouping tasks  
✅ Rescue blocks for error handling  
✅ Always blocks for cleanup  
✅ Assertions for validation  
✅ Retry logic with until  
✅ Custom failure conditions  
✅ Rollback patterns  

## Next Steps

Move on to [Exercise 18: Tags and Task Control](../exercise-18/README.md) to learn selective task execution.
