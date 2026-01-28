# Exercise 12: Variable Files

## Learning Objectives

- Store variables in external files
- Use vars_files directive
- Organize variables by environment
- Include variable files dynamically

## Prerequisites

- Completed Exercise 11
- Understanding of variable basics

## Steps

### Step 1: Create Variable Files

Create separate files for different variable sets:

```bash
# Create directory for variable files
mkdir -p /tmp/vars-files

# Create development variables
cat > /tmp/vars-files/dev.yml << 'EOF'
---
environment: development
debug_mode: true
app_port: 8080
database_host: localhost
database_port: 5432
database_name: myapp_dev
max_connections: 10
log_level: DEBUG
EOF

# Create production variables
cat > /tmp/vars-files/prod.yml << 'EOF'
---
environment: production
debug_mode: false
app_port: 80
database_host: db.example.com
database_port: 5432
database_name: myapp_prod
max_connections: 100
log_level: ERROR
EOF

# Create staging variables
cat > /tmp/vars-files/staging.yml << 'EOF'
---
environment: staging
debug_mode: true
app_port: 8080
database_host: staging-db.example.com
database_port: 5432
database_name: myapp_staging
max_connections: 50
log_level: INFO
EOF
```

### Step 2: Use vars_files

Create a playbook that uses external variable files:

```bash
cat > use-vars-files.yml << 'EOF'
---
- name: Using Variable Files
  hosts: localhost
  connection: local
  vars_files:
    - /tmp/vars-files/dev.yml
  
  tasks:
    - name: Display environment configuration
      debug:
        msg: |
          Environment: {{ environment }}
          Debug Mode: {{ debug_mode }}
          Application Port: {{ app_port }}
          Database: {{ database_host }}:{{ database_port }}/{{ database_name }}
          Max Connections: {{ max_connections }}
          Log Level: {{ log_level }}
    
    - name: Create configuration file
      copy:
        content: |
          # Application Configuration
          # Environment: {{ environment }}
          
          [application]
          debug={{ debug_mode | lower }}
          port={{ app_port }}
          log_level={{ log_level }}
          
          [database]
          host={{ database_host }}
          port={{ database_port }}
          name={{ database_name }}
          max_connections={{ max_connections }}
        dest: "/tmp/app-{{ environment }}.conf"
    
    - name: Display config file location
      debug:
        msg: "Configuration created at: /tmp/app-{{ environment }}.conf"
EOF
```

### Step 3: Test with Different Environments

```bash
# Test with dev variables (default in playbook)
ansible-playbook use-vars-files.yml

# View the created config
cat /tmp/app-development.conf
```

### Step 4: Dynamic Variable File Loading

Create a playbook that loads different variable files:

```bash
cat > dynamic-vars.yml << 'EOF'
---
- name: Dynamic Variable File Loading
  hosts: localhost
  connection: local
  vars:
    env: "{{ target_env | default('dev') }}"
  vars_files:
    - "/tmp/vars-files/{{ env }}.yml"
  
  tasks:
    - name: Show loaded environment
      debug:
        msg: "Loaded {{ environment }} environment configuration"
    
    - name: Display key settings
      debug:
        msg: |
          Port: {{ app_port }}
          Database: {{ database_name }}
          Debug: {{ debug_mode }}
    
    - name: Create environment-specific directory
      file:
        path: "/tmp/deploy/{{ environment }}"
        state: directory
    
    - name: Deploy configuration
      copy:
        content: |
          # {{ environment | upper }} Configuration
          Environment={{ environment }}
          Port={{ app_port }}
          Database={{ database_host }}:{{ database_port }}
          Debug={{ debug_mode }}
          LogLevel={{ log_level }}
        dest: "/tmp/deploy/{{ environment }}/config.ini"
EOF
```

### Step 5: Test Dynamic Loading

```bash
# Load dev environment
ansible-playbook dynamic-vars.yml -e "target_env=dev"

# Load staging environment
ansible-playbook dynamic-vars.yml -e "target_env=staging"

# Load production environment
ansible-playbook dynamic-vars.yml -e "target_env=prod"

# Check created files
ls -la /tmp/deploy/*/
```

### Step 6: Multiple Variable Files

Create a playbook using multiple variable files:

```bash
# Create common variables file
cat > /tmp/vars-files/common.yml << 'EOF'
---
app_name: MyApplication
app_version: 2.0.0
company_name: Acme Corp
support_email: support@example.com
timezone: UTC
backup_enabled: true
monitoring_enabled: true
EOF

# Create secrets file (in practice, use ansible-vault)
cat > /tmp/vars-files/secrets.yml << 'EOF'
---
api_key: abc123def456
database_password: SecurePassword123
jwt_secret: super-secret-key
EOF

# Create playbook using multiple files
cat > multiple-vars-files.yml << 'EOF'
---
- name: Multiple Variable Files
  hosts: localhost
  connection: local
  vars_files:
    - /tmp/vars-files/common.yml
    - /tmp/vars-files/dev.yml
    - /tmp/vars-files/secrets.yml
  
  tasks:
    - name: Display combined configuration
      debug:
        msg: |
          Application: {{ app_name }} v{{ app_version }}
          Company: {{ company_name }}
          Environment: {{ environment }}
          Port: {{ app_port }}
          Database: {{ database_name }}
    
    - name: Create complete configuration
      copy:
        content: |
          # {{ app_name }} Configuration
          # Version: {{ app_version }}
          # Environment: {{ environment }}
          
          [application]
          name={{ app_name }}
          version={{ app_version }}
          port={{ app_port }}
          debug={{ debug_mode }}
          log_level={{ log_level }}
          
          [company]
          name={{ company_name }}
          support_email={{ support_email }}
          timezone={{ timezone }}
          
          [database]
          host={{ database_host }}
          port={{ database_port }}
          name={{ database_name }}
          password={{ database_password }}
          
          [features]
          backup_enabled={{ backup_enabled }}
          monitoring_enabled={{ monitoring_enabled }}
          
          [secrets]
          api_key={{ api_key }}
          jwt_secret={{ jwt_secret }}
        dest: /tmp/complete-config.ini
        mode: '0600'  # Secure file with secrets
    
    - name: Verify configuration created
      stat:
        path: /tmp/complete-config.ini
      register: config_stat
    
    - name: Display config info
      debug:
        msg: |
          Config file created: {{ config_stat.stat.exists }}
          File size: {{ config_stat.stat.size }} bytes
          Permissions: {{ config_stat.stat.mode }}
EOF
```

### Step 7: Test Multiple Variable Files

```bash
ansible-playbook multiple-vars-files.yml

# View the complete config (be careful, contains "secrets")
cat /tmp/complete-config.ini
```

### Step 8: Include Variables Dynamically in Tasks

Create a playbook that includes variables during execution:

```bash
cat > include-vars-task.yml << 'EOF'
---
- name: Include Variables in Tasks
  hosts: localhost
  connection: local
  
  tasks:
    - name: Prompt for environment (simulated with default)
      set_fact:
        selected_env: "{{ deploy_env | default('dev') }}"
    
    - name: Load environment variables
      include_vars:
        file: "/tmp/vars-files/{{ selected_env }}.yml"
    
    - name: Load common variables
      include_vars:
        file: /tmp/vars-files/common.yml
    
    - name: Display loaded configuration
      debug:
        msg: |
          Application: {{ app_name }}
          Environment: {{ environment }}
          Port: {{ app_port }}
          Debug: {{ debug_mode }}
    
    - name: Load variables from directory
      include_vars:
        dir: /tmp/vars-files
        files_matching: "common.yml"
    
    - name: Create deployment summary
      copy:
        content: |
          Deployment Summary
          ==================
          Application: {{ app_name }} v{{ app_version }}
          Environment: {{ environment }}
          Timestamp: {{ ansible_date_time.iso8601 }}
          
          Configuration:
          - Port: {{ app_port }}
          - Debug: {{ debug_mode }}
          - Log Level: {{ log_level }}
          - Database: {{ database_host }}/{{ database_name }}
        dest: "/tmp/deployment-{{ environment }}-summary.txt"
EOF
```

### Step 9: Test Include Vars

```bash
# Test with different environments
ansible-playbook include-vars-task.yml -e "deploy_env=dev"
ansible-playbook include-vars-task.yml -e "deploy_env=staging"
ansible-playbook include-vars-task.yml -e "deploy_env=prod"

# View summaries
cat /tmp/deployment-*-summary.txt
```

### Step 10: Variable File Organization

Create an organized structure:

```bash
# Create organized variable structure
mkdir -p /tmp/organized-vars/{group_vars,host_vars,environments}

# Group variables
cat > /tmp/organized-vars/group_vars/webservers.yml << 'EOF'
---
http_port: 80
https_port: 443
max_connections: 100
enable_ssl: true
EOF

cat > /tmp/organized-vars/group_vars/databases.yml << 'EOF'
---
db_port: 5432
max_connections: 200
backup_schedule: "0 2 * * *"
EOF

# Environment-specific
cat > /tmp/organized-vars/environments/production.yml << 'EOF'
---
environment: production
replica_count: 3
auto_scaling: true
monitoring: true
EOF

# Create playbook using organized structure
cat > organized-vars.yml << 'EOF'
---
- name: Organized Variable Structure
  hosts: localhost
  connection: local
  vars_files:
    - /tmp/organized-vars/group_vars/webservers.yml
    - /tmp/organized-vars/environments/production.yml
  
  tasks:
    - name: Display webserver configuration
      debug:
        msg: |
          Environment: {{ environment }}
          HTTP Port: {{ http_port }}
          HTTPS Port: {{ https_port }}
          SSL Enabled: {{ enable_ssl }}
          Max Connections: {{ max_connections }}
          Replicas: {{ replica_count }}
    
    - name: Create organized config
      copy:
        content: |
          # Webserver Configuration
          # Environment: {{ environment }}
          
          [http]
          port={{ http_port }}
          ssl_port={{ https_port }}
          ssl_enabled={{ enable_ssl | lower }}
          
          [performance]
          max_connections={{ max_connections }}
          
          [scaling]
          replicas={{ replica_count }}
          auto_scaling={{ auto_scaling | lower }}
          monitoring={{ monitoring | lower }}
        dest: /tmp/organized-config.ini
EOF
```

### Step 11: Test Organized Structure

```bash
ansible-playbook organized-vars.yml
cat /tmp/organized-config.ini
```

## Expected Results

### Loading Variable File

```
TASK [Display environment configuration] *************************************
ok: [localhost] => {
    "msg": "Environment: development\nDebug Mode: True\nApplication Port: 8080\n..."
}
```

### Dynamic File Loading

```
TASK [Show loaded environment] ***********************************************
ok: [localhost] => {
    "msg": "Loaded production environment configuration"
}
```

## Variable File Best Practices

### Directory Structure
```
inventory/
├── group_vars/
│   ├── all.yml
│   ├── webservers.yml
│   └── databases.yml
├── host_vars/
│   ├── host1.yml
│   └── host2.yml
└── environments/
    ├── dev.yml
    ├── staging.yml
    └── prod.yml
```

### File Naming
- Use `.yml` or `.yaml` extensions
- Use lowercase with hyphens: `my-vars.yml`
- Environment-specific: `prod.yml`, `dev.yml`
- Group-specific: `webservers.yml`

## Troubleshooting

### Problem: Variable file not found

**Solution:** Check path and use absolute paths
```yaml
vars_files:
  - "{{ playbook_dir }}/vars/common.yml"
```

### Problem: Variables not overriding

**Solution:** Check variable precedence and file order

### Problem: YAML syntax error

**Solution:** Validate YAML syntax
```bash
ansible-playbook playbook.yml --syntax-check
```

## Additional Exercises

1. Create variable files for 3 different environments
2. Organize variables by server role
3. Build a dynamic variable loader based on hostname
4. Create a variable file hierarchy with overrides
5. Practice variable precedence with multiple sources

## What You Learned

✅ Creating external variable files  
✅ Using vars_files directive  
✅ Loading variables dynamically based on conditions  
✅ Using multiple variable files  
✅ Include_vars task for runtime variable loading  
✅ Organizing variables by environment and role  
✅ Variable file best practices  
✅ Managing secrets in variable files  

## Next Steps

Move on to [Exercise 13: Jinja2 Templates](../exercise-13/README.md) to learn about creating dynamic configuration files.
