# Exercise 18: Tags and Task Control

## Learning Objectives

- Use tags for selective execution
- Control task execution flow
- Implement tagging strategies
- Skip and limit task execution

## Prerequisites

- Completed Exercise 17
- Understanding of playbook structure

## Steps

### Step 1: Basic Tags

Create a playbook with basic tags:

```bash
cat > basic-tags.yml << 'EOF'
---
- name: Basic Tags Example
  hosts: localhost
  connection: local
  
  tasks:
    - name: Install packages
      debug:
        msg: "Installing packages..."
      tags:
        - install
        - packages
    
    - name: Configure application
      debug:
        msg: "Configuring application..."
      tags:
        - configure
        - config
    
    - name: Start services
      debug:
        msg: "Starting services..."
      tags:
        - start
        - services
    
    - name: Run tests
      debug:
        msg: "Running tests..."
      tags:
        - test
        - verify
    
    - name: Always run this task
      debug:
        msg: "This task always runs"
      tags:
        - always
    
    - name: Never run this by default
      debug:
        msg: "This task is tagged never"
      tags:
        - never
        - dangerous
EOF
```

### Step 2: Test Tag Execution

```bash
# Run all tasks
ansible-playbook basic-tags.yml

# Run only install tasks
ansible-playbook basic-tags.yml --tags install

# Run multiple tags
ansible-playbook basic-tags.yml --tags "install,configure"

# Skip specific tags
ansible-playbook basic-tags.yml --skip-tags test

# Run only never-tagged tasks
ansible-playbook basic-tags.yml --tags never

# List all tags
ansible-playbook basic-tags.yml --list-tags
```

### Step 3: Complex Tagging Strategy

Create a deployment playbook with comprehensive tags:

```bash
cat > deployment-tags.yml << 'EOF'
---
- name: Application Deployment with Tags
  hosts: localhost
  connection: local
  
  tasks:
    # Pre-deployment
    - name: Backup current version
      copy:
        content: "Backup {{ ansible_date_time.epoch }}\n"
        dest: /tmp/backup.txt
      tags:
        - predeploy
        - backup
        - safety
    
    - name: Stop application
      debug:
        msg: "Stopping application..."
      tags:
        - predeploy
        - stop
    
    # Installation
    - name: Download application
      debug:
        msg: "Downloading application..."
      tags:
        - deploy
        - install
        - download
    
    - name: Extract application
      debug:
        msg: "Extracting files..."
      tags:
        - deploy
        - install
        - extract
    
    # Configuration
    - name: Copy configuration files
      copy:
        content: |
          app_port=8080
          debug=false
        dest: /tmp/app-config.ini
      tags:
        - deploy
        - configure
        - config
    
    - name: Set permissions
      file:
        path: /tmp/app-config.ini
        mode: '0644'
      tags:
        - deploy
        - configure
        - permissions
    
    # Database
    - name: Run database migrations
      debug:
        msg: "Running migrations..."
      tags:
        - deploy
        - database
        - migrations
    
    - name: Seed database
      debug:
        msg: "Seeding database..."
      tags:
        - deploy
        - database
        - seed
        - never  # Don't run by default
    
    # Post-deployment
    - name: Start application
      debug:
        msg: "Starting application..."
      tags:
        - postdeploy
        - start
    
    - name: Health check
      debug:
        msg: "Running health check..."
      tags:
        - postdeploy
        - verify
        - health
    
    - name: Warm up cache
      debug:
        msg: "Warming up cache..."
      tags:
        - postdeploy
        - optimization
    
    # Cleanup
    - name: Remove temporary files
      debug:
        msg: "Cleaning up..."
      tags:
        - cleanup
        - always
    
    # Rollback (never runs unless explicitly requested)
    - name: Rollback to previous version
      debug:
        msg: "Rolling back..."
      tags:
        - never
        - rollback
EOF
```

### Step 4: Test Deployment Tags

```bash
# Full deployment
ansible-playbook deployment-tags.yml

# Only configuration changes
ansible-playbook deployment-tags.yml --tags configure

# Install without starting
ansible-playbook deployment-tags.yml --tags install

# Database operations only
ansible-playbook deployment-tags.yml --tags database

# Skip database operations
ansible-playbook deployment-tags.yml --skip-tags database

# Rollback
ansible-playbook deployment-tags.yml --tags rollback

# List available tags
ansible-playbook deployment-tags.yml --list-tags
```

### Step 5: Tags with Blocks and Roles

Create a playbook combining tags with blocks:

```bash
cat > tags-with-blocks.yml << 'EOF'
---
- name: Tags with Blocks
  hosts: localhost
  connection: local
  
  tasks:
    - name: Web server setup block
      tags:
        - webserver
        - web
      block:
        - name: Install web server
          debug:
            msg: "Installing web server..."
          tags:
            - install
        
        - name: Configure web server
          debug:
            msg: "Configuring web server..."
          tags:
            - configure
        
        - name: Start web server
          debug:
            msg: "Starting web server..."
          tags:
            - start
    
    - name: Database setup block
      tags:
        - database
        - db
      block:
        - name: Install database
          debug:
            msg: "Installing database..."
          tags:
            - install
        
        - name: Configure database
          debug:
            msg: "Configuring database..."
          tags:
            - configure
        
        - name: Initialize database
          debug:
            msg: "Initializing database..."
          tags:
            - init
    
    - name: Monitoring setup
      tags:
        - monitoring
        - optional
      block:
        - name: Install monitoring agent
          debug:
            msg: "Installing monitoring..."
          tags:
            - install
        
        - name: Configure alerts
          debug:
            msg: "Configuring alerts..."
          tags:
            - configure
EOF
```

### Step 6: Test Block Tags

```bash
# Only web server
ansible-playbook tags-with-blocks.yml --tags webserver

# Only database
ansible-playbook tags-with-blocks.yml --tags database

# All install tasks across blocks
ansible-playbook tags-with-blocks.yml --tags install

# Web server config only
ansible-playbook tags-with-blocks.yml --tags "webserver,configure"
```

### Step 7: Dynamic Task Control

Create a playbook with dynamic task control:

```bash
cat > task-control.yml << 'EOF'
---
- name: Dynamic Task Control
  hosts: localhost
  connection: local
  vars:
    enable_feature_a: true
    enable_feature_b: false
    run_mode: "production"  # development, staging, production
  
  tasks:
    - name: Feature A tasks
      tags:
        - feature-a
        - features
      block:
        - name: Setup Feature A
          debug:
            msg: "Setting up Feature A"
          when: enable_feature_a
        
        - name: Configure Feature A
          debug:
            msg: "Configuring Feature A"
          when: enable_feature_a
    
    - name: Feature B tasks
      tags:
        - feature-b
        - features
      block:
        - name: Setup Feature B
          debug:
            msg: "Setting up Feature B"
          when: enable_feature_b
        
        - name: Configure Feature B
          debug:
            msg: "Configuring Feature B"
          when: enable_feature_b
    
    - name: Development tasks
      debug:
        msg: "Running development-specific tasks"
      tags:
        - development
        - dev
      when: run_mode == "development"
    
    - name: Production tasks
      debug:
        msg: "Running production-specific tasks"
      tags:
        - production
        - prod
      when: run_mode == "production"
    
    - name: Debug tasks
      debug:
        msg: |
          Features enabled:
          - Feature A: {{ enable_feature_a }}
          - Feature B: {{ enable_feature_b }}
          Run mode: {{ run_mode }}
      tags:
        - debug
        - info
EOF
```

### Step 8: Test Dynamic Control

```bash
# Run with defaults
ansible-playbook task-control.yml --tags features

# Enable Feature B
ansible-playbook task-control.yml --tags feature-b -e "enable_feature_b=true"

# Development mode
ansible-playbook task-control.yml --tags development -e "run_mode=development"

# Show debug info
ansible-playbook task-control.yml --tags debug
```

### Step 9: Tagged Import and Include

Create a playbook with tagged imports:

```bash
# Create separate task files
cat > /tmp/install-tasks.yml << 'EOF'
---
- name: Install dependencies
  debug:
    msg: "Installing dependencies..."

- name: Install application
  debug:
    msg: "Installing application..."
EOF

cat > /tmp/configure-tasks.yml << 'EOF'
---
- name: Generate configuration
  debug:
    msg: "Generating configuration..."

- name: Apply configuration
  debug:
    msg: "Applying configuration..."
EOF

# Main playbook with imports
cat > import-with-tags.yml << 'EOF'
---
- name: Import Tasks with Tags
  hosts: localhost
  connection: local
  
  tasks:
    - name: Pre-installation check
      debug:
        msg: "Checking prerequisites..."
      tags:
        - always
    
    - name: Import installation tasks
      import_tasks: /tmp/install-tasks.yml
      tags:
        - install
    
    - name: Import configuration tasks
      include_tasks: /tmp/configure-tasks.yml
      tags:
        - configure
    
    - name: Post-installation tasks
      debug:
        msg: "Finalizing installation..."
      tags:
        - finalize
EOF
```

### Step 10: Test Tagged Imports

```bash
# Run only installation
ansible-playbook import-with-tags.yml --tags install

# Run only configuration
ansible-playbook import-with-tags.yml --tags configure

# List all tasks
ansible-playbook import-with-tags.yml --list-tasks
```

### Step 11: Complete Tagging Example

Create a comprehensive example:

```bash
cat > complete-tags-example.yml << 'EOF'
---
- name: Complete Tagging Strategy
  hosts: localhost
  connection: local
  vars:
    deployment_env: production
  
  tasks:
    # Initialization (always runs)
    - name: Initialize deployment
      debug:
        msg: "Initializing deployment for {{ deployment_env }}"
      tags: always
    
    # Preparation phase
    - name: Preparation phase
      tags:
        - phase-1
        - preparation
      block:
        - name: Validate prerequisites
          debug:
            msg: "Validating prerequisites..."
          tags: validate
        
        - name: Backup current state
          copy:
            content: "Backup {{ ansible_date_time.iso8601 }}\n"
            dest: /tmp/state-backup.txt
          tags: backup
    
    # Installation phase
    - name: Installation phase
      tags:
        - phase-2
        - installation
      block:
        - name: Download packages
          debug:
            msg: "Downloading packages..."
          tags: download
        
        - name: Install packages
          debug:
            msg: "Installing packages..."
          tags: install
        
        - name: Verify installation
          debug:
            msg: "Verifying installation..."
          tags: verify
    
    # Configuration phase
    - name: Configuration phase
      tags:
        - phase-3
        - configuration
      block:
        - name: Generate configs
          copy:
            content: |
              environment={{ deployment_env }}
              timestamp={{ ansible_date_time.epoch }}
            dest: /tmp/app-{{ deployment_env }}.conf
          tags: generate
        
        - name: Apply configs
          debug:
            msg: "Applying configurations..."
          tags: apply
        
        - name: Test configs
          debug:
            msg: "Testing configurations..."
          tags: test
    
    # Deployment phase
    - name: Deployment phase
      tags:
        - phase-4
        - deployment
      block:
        - name: Deploy application
          debug:
            msg: "Deploying application..."
          tags: deploy
        
        - name: Start services
          debug:
            msg: "Starting services..."
          tags: start
        
        - name: Health check
          debug:
            msg: "Running health check..."
          tags: health
    
    # Finalization (always runs)
    - name: Cleanup and logging
      copy:
        content: |
          Deployment Summary
          ==================
          Environment: {{ deployment_env }}
          Timestamp: {{ ansible_date_time.iso8601 }}
          Status: Completed
        dest: /tmp/deployment-summary.txt
      tags: always
    
    # Maintenance tasks (never run automatically)
    - name: Maintenance tasks
      tags:
        - never
        - maintenance
      block:
        - name: Clean cache
          debug:
            msg: "Cleaning cache..."
        
        - name: Optimize database
          debug:
            msg: "Optimizing database..."
        
        - name: Update indices
          debug:
            msg: "Updating indices..."
    
    # Emergency rollback (never run automatically)
    - name: Emergency rollback
      debug:
        msg: "Performing emergency rollback..."
      tags:
        - never
        - emergency
        - rollback
EOF
```

### Step 12: Test Complete Example

```bash
# Full deployment
ansible-playbook complete-tags-example.yml

# Only specific phase
ansible-playbook complete-tags-example.yml --tags phase-2

# Multiple phases
ansible-playbook complete-tags-example.yml --tags "phase-1,phase-2"

# Skip a phase
ansible-playbook complete-tags-example.yml --skip-tags phase-3

# Run maintenance
ansible-playbook complete-tags-example.yml --tags maintenance

# Emergency rollback
ansible-playbook complete-tags-example.yml --tags emergency

# List all tags
ansible-playbook complete-tags-example.yml --list-tags

# List all tasks
ansible-playbook complete-tags-example.yml --list-tasks

# View deployment summary
cat /tmp/deployment-summary.txt
```

## Expected Results

### Tag Execution

```
TASK [Install packages] ******************************************************
ok: [localhost] => {
    "msg": "Installing packages..."
}

PLAY RECAP *******************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0
```

### List Tags Output

```
playbook: basic-tags.yml

  play #1 (localhost): Basic Tags Example	TAGS: []
      TASK TAGS: [always, config, configure, dangerous, install, never, packages, 
                 services, start, test, verify]
```

## Tag Best Practices

### Naming Conventions
```yaml
tags:
  - install          # Action
  - webserver        # Component
  - phase-1          # Phase
  - production       # Environment
```

### Common Tag Patterns
- `always` - Always run
- `never` - Never run by default
- `install`, `configure`, `start`
- Component names: `database`, `webserver`
- Phases: `phase-1`, `phase-2`
- Environments: `dev`, `prod`

## Troubleshooting

### Problem: Tags not working

**Solution:** Check tag spelling and --list-tags
```bash
ansible-playbook playbook.yml --list-tags
```

### Problem: Always tag not running

**Solution:** Verify always tag is correctly applied

### Problem: Multiple tags confusion

**Solution:** Use --list-tasks to see what will run
```bash
ansible-playbook playbook.yml --tags install --list-tasks
```

## Additional Exercises

1. Create a deployment with 5 distinct phases using tags
2. Implement environment-specific tags (dev/stage/prod)
3. Build a maintenance playbook with never-tagged tasks
4. Create tag hierarchy for complex deployments
5. Design rollback procedures with emergency tags

## What You Learned

✅ Using tags for selective execution  
✅ Special tags (always, never)  
✅ Tagging strategies and patterns  
✅ Tags with blocks and roles  
✅ Multiple tag selection  
✅ Skip tags functionality  
✅ Listing tags and tasks  
✅ Phase-based execution with tags  

## Next Steps

Move on to [Exercise 19: Dynamic Inventory](../exercise-19/README.md) to learn about generating inventories dynamically.
