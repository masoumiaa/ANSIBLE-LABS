# Exercise 06: Working with Handlers

## Learning Objectives

- Understand the purpose of handlers
- Notify handlers from tasks
- Use handlers for service management
- Control handler execution

## Prerequisites

- Completed Exercise 05
- Understanding of service management concepts

## Steps

### Step 1: Basic Handler Example

Create a playbook with a simple handler:

```bash
cat > basic-handler.yml << 'EOF'
---
- name: Basic Handler Example
  hosts: localhost
  connection: local
  
  tasks:
    - name: Create a configuration file
      copy:
        content: |
          # Application Configuration
          app_name: TestApp
          version: 1.0
        dest: /tmp/app.conf
      notify: Display config change
  
  handlers:
    - name: Display config change
      debug:
        msg: "Configuration file has been modified!"
EOF
```

### Step 2: Run the Basic Example

```bash
# First run - handler will execute
ansible-playbook basic-handler.yml

# Second run - no changes, handler won't execute
ansible-playbook basic-handler.yml
```

### Step 3: Multiple Handlers

Create a playbook with multiple handlers:

```bash
cat > multiple-handlers.yml << 'EOF'
---
- name: Multiple Handlers Example
  hosts: localhost
  connection: local
  
  tasks:
    - name: Create config directory
      file:
        path: /tmp/myapp
        state: directory
      notify:
        - Log directory creation
        - Update timestamp
    
    - name: Create main configuration
      copy:
        content: |
          [main]
          enabled=true
        dest: /tmp/myapp/main.conf
      notify:
        - Log config change
        - Update timestamp
    
    - name: Create secondary configuration
      copy:
        content: |
          [secondary]
          backup=true
        dest: /tmp/myapp/secondary.conf
      notify: Log config change
  
  handlers:
    - name: Log directory creation
      debug:
        msg: "Directory structure has been created"
    
    - name: Log config change
      debug:
        msg: "Configuration files have been updated"
    
    - name: Update timestamp
      copy:
        content: "Last updated: {{ ansible_date_time.iso8601 }}\n"
        dest: /tmp/myapp/last-update.txt
      notify: Display update time
    
    - name: Display update time
      debug:
        msg: "Timestamp file updated"
EOF
```

### Step 4: Execute Multiple Handlers

```bash
ansible-playbook multiple-handlers.yml
```

### Step 5: Service Restart Handler

Create a realistic example with file configuration:

```bash
cat > service-handler.yml << 'EOF'
---
- name: Service Management with Handlers
  hosts: localhost
  connection: local
  
  tasks:
    - name: Create service directory
      file:
        path: /tmp/myservice
        state: directory
    
    - name: Deploy application configuration
      copy:
        content: |
          # Service Configuration
          port=8080
          max_connections=100
          timeout=30
        dest: /tmp/myservice/service.conf
      notify: Restart service
    
    - name: Deploy application code
      copy:
        content: |
          #!/bin/bash
          echo "Service running with config from service.conf"
        dest: /tmp/myservice/app.sh
        mode: '0755'
      notify: Restart service
    
    - name: Create service status file
      copy:
        content: "status=running\n"
        dest: /tmp/myservice/status.txt
  
  handlers:
    - name: Restart service
      debug:
        msg: "Service would be restarted here (simulated)"
      notify: Verify service
    
    - name: Verify service
      debug:
        msg: "Service restart verified successfully"
EOF
```

### Step 6: Run Service Handler Example

```bash
# Run once - handlers execute
ansible-playbook service-handler.yml

# Run again - no changes, handlers don't execute
ansible-playbook service-handler.yml

# Make a change and run again
echo "port=9090" >> /tmp/myservice/service.conf
ansible-playbook service-handler.yml
```

### Step 7: Force Handler Execution

Create a playbook demonstrating forced handler execution:

```bash
cat > force-handlers.yml << 'EOF'
---
- name: Force Handlers Example
  hosts: localhost
  connection: local
  force_handlers: yes
  
  tasks:
    - name: Task that notifies handler
      copy:
        content: "Initial content\n"
        dest: /tmp/test-force.txt
      notify: Cleanup handler
    
    - name: Task that might fail
      command: /bin/false
      ignore_errors: yes
  
  handlers:
    - name: Cleanup handler
      debug:
        msg: "Cleanup executed even though a task failed (force_handlers: yes)"
EOF
```

### Step 8: Test Force Handlers

```bash
ansible-playbook force-handlers.yml
```

### Step 9: Handler with Listen

Create handlers that respond to generic notifications:

```bash
cat > listen-handlers.yml << 'EOF'
---
- name: Handlers with Listen Keyword
  hosts: localhost
  connection: local
  
  tasks:
    - name: Update web configuration
      copy:
        content: "web_port=80\n"
        dest: /tmp/web.conf
      notify: restart web services
    
    - name: Update database configuration
      copy:
        content: "db_port=5432\n"
        dest: /tmp/db.conf
      notify: restart database services
    
    - name: Update cache configuration
      copy:
        content: "cache_size=128M\n"
        dest: /tmp/cache.conf
      notify:
        - restart web services
        - clear cache
  
  handlers:
    - name: Restart nginx
      debug:
        msg: "Nginx restarted"
      listen: restart web services
    
    - name: Restart apache
      debug:
        msg: "Apache restarted"
      listen: restart web services
    
    - name: Restart postgresql
      debug:
        msg: "PostgreSQL restarted"
      listen: restart database services
    
    - name: Clear application cache
      debug:
        msg: "Cache cleared"
      listen: clear cache
EOF
```

### Step 10: Run Listen Example

```bash
ansible-playbook listen-handlers.yml
```

## Expected Results

### Handler Execution Output

```
TASK [Create a configuration file] *******************************************
changed: [localhost]

RUNNING HANDLER [Display config change] **************************************
ok: [localhost] => {
    "msg": "Configuration file has been modified!"
}

PLAY RECAP *******************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0
```

### No Changes - No Handler

```
TASK [Create a configuration file] *******************************************
ok: [localhost]

PLAY RECAP *******************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0
```

## Key Concepts

### Handler Execution Rules

1. **Handlers run at the end of play** - After all tasks complete
2. **Handlers run only if notified** - Only when a task reports "changed"
3. **Handlers run once** - Even if notified multiple times
4. **Handlers run in order** - In the order they're defined (not notified)

### Handler Syntax

```yaml
tasks:
  - name: Task
    module: ...
    notify: Handler Name

handlers:
  - name: Handler Name
    module: ...
```

### Listen Keyword

```yaml
handlers:
  - name: Specific handler
    module: ...
    listen: generic notification
```

## Troubleshooting

### Problem: Handler doesn't run

**Solution:** Check if task actually changed
```bash
ansible-playbook playbook.yml -v
```

### Problem: Handler runs too early

**Solution:** Handlers always run at the end; use `meta: flush_handlers` to force early execution
```yaml
- name: Force handlers to run now
  meta: flush_handlers
```

### Problem: Need handler to run even on failure

**Solution:** Use `force_handlers: yes` in play
```yaml
- name: Play name
  hosts: all
  force_handlers: yes
```

## Additional Exercises

1. Create a playbook with 5 tasks that notify 3 different handlers
2. Implement a handler chain (one handler notifies another)
3. Use `meta: flush_handlers` to run handlers mid-play
4. Create handlers with the `listen` keyword for grouped notifications
5. Build a realistic web server config update with service restart

## What You Learned

✅ Purpose and benefits of handlers  
✅ Notifying handlers from tasks  
✅ Handler execution order and rules  
✅ Multiple handlers for one task  
✅ Handler chaining (handlers notifying handlers)  
✅ Using `listen` keyword for grouped handlers  
✅ Forcing handler execution with `force_handlers`  
✅ Flushing handlers with `meta: flush_handlers`  

## Next Steps

Move on to [Exercise 07: Conditional Execution](../exercise-07/README.md) to learn how to control task execution with conditions.
