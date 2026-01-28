# Exercise 14: Facts and Filters

## Learning Objectives

- Gather and use system facts
- Create custom facts
- Apply advanced filters
- Filter and transform data

## Prerequisites

- Completed Exercise 13
- Understanding of Jinja2 templates

## Steps

### Step 1: Exploring System Facts

Create a playbook to explore available facts:

```bash
cat > explore-facts.yml << 'EOF'
---
- name: Explore System Facts
  hosts: localhost
  connection: local
  gather_facts: yes
  
  tasks:
    - name: Display all facts (limited output)
      debug:
        var: ansible_facts
      tags: never  # Skip by default, run with --tags show_all
    
    - name: Display system information
      debug:
        msg: |
          Hostname: {{ ansible_hostname }}
          FQDN: {{ ansible_fqdn }}
          OS: {{ ansible_distribution }} {{ ansible_distribution_version }}
          Kernel: {{ ansible_kernel }}
          Architecture: {{ ansible_architecture }}
    
    - name: Display hardware information
      debug:
        msg: |
          CPU Count: {{ ansible_processor_count }}
          CPU Cores: {{ ansible_processor_cores }}
          Total Memory: {{ ansible_memtotal_mb }} MB
          Free Memory: {{ ansible_memfree_mb }} MB
          Swap Total: {{ ansible_swaptotal_mb }} MB
    
    - name: Display network information
      debug:
        msg: |
          Default IPv4: {{ ansible_default_ipv4.address | default('N/A') }}
          Gateway: {{ ansible_default_ipv4.gateway | default('N/A') }}
          All IPs: {{ ansible_all_ipv4_addresses | join(', ') }}
    
    - name: Display Python information
      debug:
        msg: |
          Python Version: {{ ansible_python_version }}
          Python Executable: {{ ansible_python.executable }}
    
    - name: Display date/time information
      debug:
        msg: |
          Date: {{ ansible_date_time.date }}
          Time: {{ ansible_date_time.time }}
          ISO8601: {{ ansible_date_time.iso8601 }}
          Epoch: {{ ansible_date_time.epoch }}
          Timezone: {{ ansible_date_time.tz }}
    
    - name: Create facts report
      copy:
        content: |
          System Facts Report
          ===================
          Generated: {{ ansible_date_time.iso8601 }}
          
          System Information:
          - Hostname: {{ ansible_hostname }}
          - OS: {{ ansible_distribution }} {{ ansible_distribution_version }}
          - Kernel: {{ ansible_kernel }}
          - Architecture: {{ ansible_architecture }}
          
          Hardware:
          - CPUs: {{ ansible_processor_count }}
          - Cores: {{ ansible_processor_cores }}
          - Memory: {{ ansible_memtotal_mb }} MB
          
          Network:
          - IP: {{ ansible_default_ipv4.address | default('N/A') }}
          - Gateway: {{ ansible_default_ipv4.gateway | default('N/A') }}
        dest: /tmp/facts-report.txt
EOF
```

### Step 2: Run Facts Exploration

```bash
ansible-playbook explore-facts.yml
cat /tmp/facts-report.txt
```

### Step 3: Custom Facts

Create custom facts for your system:

```bash
# Create custom facts directory
mkdir -p /tmp/ansible-custom-facts

# Create a custom fact file
cat > /tmp/ansible-custom-facts/app_info.fact << 'EOF'
#!/bin/bash
# Custom fact script

cat << JSON
{
  "application": {
    "name": "CustomApp",
    "version": "3.2.1",
    "environment": "production",
    "installed_date": "2026-01-01"
  },
  "database": {
    "type": "postgresql",
    "version": "14.0",
    "port": 5432
  },
  "monitoring": {
    "enabled": true,
    "endpoint": "metrics.example.com"
  }
}
JSON
EOF

chmod +x /tmp/ansible-custom-facts/app_info.fact

# Create playbook to use custom facts
cat > custom-facts.yml << 'EOF'
---
- name: Custom Facts
  hosts: localhost
  connection: local
  gather_facts: yes
  
  tasks:
    - name: Create facts.d directory
      file:
        path: /etc/ansible/facts.d
        state: directory
        mode: '0755'
      become: yes
      ignore_errors: yes
    
    - name: Copy custom fact
      copy:
        src: /tmp/ansible-custom-facts/app_info.fact
        dest: /etc/ansible/facts.d/app_info.fact
        mode: '0755'
      become: yes
      ignore_errors: yes
    
    - name: Re-gather facts to include custom facts
      setup:
    
    - name: Display custom facts
      debug:
        var: ansible_local.app_info
      when: ansible_local.app_info is defined
    
    - name: Use custom facts
      debug:
        msg: |
          App: {{ ansible_local.app_info.application.name | default('N/A') }}
          Version: {{ ansible_local.app_info.application.version | default('N/A') }}
          Environment: {{ ansible_local.app_info.application.environment | default('N/A') }}
      when: ansible_local.app_info is defined
    
    - name: Create custom JSON fact
      copy:
        content: |
          {
            "deployment": {
              "version": "1.0.0",
              "timestamp": "{{ ansible_date_time.iso8601 }}",
              "deployed_by": "ansible"
            }
          }
        dest: /tmp/deployment.fact
    
    - name: Read custom fact
      slurp:
        src: /tmp/deployment.fact
      register: custom_fact_content
    
    - name: Parse and display custom fact
      debug:
        msg: "{{ custom_fact_content.content | b64decode | from_json }}"
EOF
```

### Step 4: Test Custom Facts

```bash
ansible-playbook custom-facts.yml
```

### Step 5: Advanced Filtering

Create a playbook demonstrating advanced filters:

```bash
cat > advanced-filters.yml << 'EOF'
---
- name: Advanced Filters
  hosts: localhost
  connection: local
  vars:
    users:
      - name: alice
        age: 30
        role: admin
        active: true
      - name: bob
        age: 25
        role: developer
        active: true
      - name: charlie
        age: 35
        role: developer
        active: false
      - name: david
        age: 28
        role: tester
        active: true
    
    numbers: [5, 2, 8, 1, 9, 3, 7]
    
    servers:
      web1: 192.168.1.10
      web2: 192.168.1.11
      db1: 192.168.1.20
      cache1: 192.168.1.30
  
  tasks:
    - name: Select attribute filter
      debug:
        msg: "Active users: {{ users | selectattr('active') | map(attribute='name') | list }}"
    
    - name: Reject attribute filter
      debug:
        msg: "Inactive users: {{ users | rejectattr('active') | map(attribute='name') | list }}"
    
    - name: Filter by role
      debug:
        msg: "Developers: {{ users | selectattr('role', 'equalto', 'developer') | map(attribute='name') | list }}"
    
    - name: Map attribute
      debug:
        msg: "All names: {{ users | map(attribute='name') | list }}"
    
    - name: Map attribute with join
      debug:
        msg: "User list: {{ users | map(attribute='name') | join(', ') }}"
    
    - name: Unique filter
      debug:
        msg: "Unique roles: {{ users | map(attribute='role') | unique | list }}"
    
    - name: Sort list
      debug:
        msg: "Sorted numbers: {{ numbers | sort }}"
    
    - name: Reverse sort
      debug:
        msg: "Reverse sorted: {{ numbers | sort(reverse=true) }}"
    
    - name: Math filters
      debug:
        msg: |
          Sum: {{ numbers | sum }}
          Min: {{ numbers | min }}
          Max: {{ numbers | max }}
    
    - name: Dictionary filters
      debug:
        msg: |
          Dict keys: {{ servers.keys() | list }}
          Dict values: {{ servers.values() | list }}
          Dict items: {{ servers.items() | list }}
    
    - name: Combine dictionaries
      debug:
        msg: "{{ servers | combine({'cache2': '192.168.1.31'}) }}"
    
    - name: JSON and YAML conversions
      debug:
        msg: |
          JSON: {{ users[0] | to_json }}
          YAML: {{ users[0] | to_yaml }}
    
    - name: Regular expression filters
      debug:
        msg: "{{ 'test-server-01' | regex_replace('-', '_') }}"
    
    - name: IP address filters
      debug:
        msg: |
          Is IP: {{ '192.168.1.10' | ipaddr }}
          Network: {{ '192.168.1.10/24' | ipaddr('network') }}
      ignore_errors: yes
    
    - name: Create filtered report
      copy:
        content: |
          Filtered Data Report
          ====================
          
          Active Users:
          {% for user in users | selectattr('active') %}
          - {{ user.name }} ({{ user.role }}, age {{ user.age }})
          {% endfor %}
          
          Users by Role:
          {% for role in users | map(attribute='role') | unique %}
          {{ role }}:
          {% for user in users | selectattr('role', 'equalto', role) %}
            - {{ user.name }}
          {% endfor %}
          {% endfor %}
          
          Statistics:
          - Total users: {{ users | length }}
          - Active users: {{ users | selectattr('active') | list | length }}
          - Average age: {{ (users | map(attribute='age') | sum / users | length) | round(1) }}
          
          Servers:
          {% for name, ip in servers.items() %}
          - {{ name }}: {{ ip }}
          {% endfor %}
        dest: /tmp/filtered-report.txt
EOF
```

### Step 6: Test Advanced Filters

```bash
ansible-playbook advanced-filters.yml
cat /tmp/filtered-report.txt
```

### Step 7: Fact Caching

Create a playbook demonstrating fact caching:

```bash
cat > fact-caching.yml << 'EOF'
---
- name: Fact Caching
  hosts: localhost
  connection: local
  gather_facts: yes
  
  tasks:
    - name: Set custom fact with set_fact
      set_fact:
        custom_variable: "Custom value"
        deployment_id: "{{ 99999999 | random | to_uuid }}"
        timestamp: "{{ ansible_date_time.epoch }}"
    
    - name: Display custom facts
      debug:
        msg: |
          Custom Variable: {{ custom_variable }}
          Deployment ID: {{ deployment_id }}
          Timestamp: {{ timestamp }}
    
    - name: Set complex fact
      set_fact:
        deployment_info:
          id: "{{ deployment_id }}"
          timestamp: "{{ ansible_date_time.iso8601 }}"
          host: "{{ ansible_hostname }}"
          user: "{{ ansible_user_id }}"
    
    - name: Display complex fact
      debug:
        var: deployment_info
    
    - name: Cache facts to file
      copy:
        content: "{{ deployment_info | to_nice_json }}"
        dest: /tmp/cached-facts.json
    
    - name: Add to existing fact
      set_fact:
        deployment_info: "{{ deployment_info | combine({'status': 'completed'}) }}"
    
    - name: Display updated fact
      debug:
        var: deployment_info
EOF
```

### Step 8: Test Fact Caching

```bash
ansible-playbook fact-caching.yml
cat /tmp/cached-facts.json
```

### Step 9: Working with Complex Data

Create a playbook for complex data manipulation:

```bash
cat > complex-data.yml << 'EOF'
---
- name: Complex Data Manipulation
  hosts: localhost
  connection: local
  gather_facts: yes
  vars:
    infrastructure:
      production:
        webservers:
          - name: web1
            ip: 192.168.1.10
            port: 80
            status: active
          - name: web2
            ip: 192.168.1.11
            port: 80
            status: active
        databases:
          - name: db1
            ip: 192.168.1.20
            port: 5432
            status: active
      staging:
        webservers:
          - name: web-staging
            ip: 192.168.2.10
            port: 8080
            status: active
  
  tasks:
    - name: Get all production servers
      debug:
        msg: "Production webservers: {{ infrastructure.production.webservers | map(attribute='name') | list }}"
    
    - name: Get all active servers
      set_fact:
        all_prod_servers: "{{ infrastructure.production.webservers + infrastructure.production.databases }}"
    
    - name: Display all production servers
      debug:
        msg: "Server: {{ item.name }}, IP: {{ item.ip }}, Port: {{ item.port }}"
      loop: "{{ all_prod_servers }}"
    
    - name: Filter active servers
      debug:
        msg: "Active servers: {{ all_prod_servers | selectattr('status', 'equalto', 'active') | map(attribute='name') | list }}"
    
    - name: Group servers by type
      set_fact:
        server_ips: "{{ all_prod_servers | map(attribute='ip') | list }}"
    
    - name: Display server IPs
      debug:
        msg: "All production IPs: {{ server_ips | join(', ') }}"
    
    - name: Create inventory from data
      copy:
        content: |
          # Generated Inventory
          # Date: {{ ansible_date_time.date }}
          
          [production_web]
          {% for server in infrastructure.production.webservers %}
          {{ server.name }} ansible_host={{ server.ip }} ansible_port={{ server.port }}
          {% endfor %}
          
          [production_db]
          {% for server in infrastructure.production.databases %}
          {{ server.name }} ansible_host={{ server.ip }} ansible_port={{ server.port }}
          {% endfor %}
          
          [staging_web]
          {% for server in infrastructure.staging.webservers %}
          {{ server.name }} ansible_host={{ server.ip }} ansible_port={{ server.port }}
          {% endfor %}
          
          [production:children]
          production_web
          production_db
        dest: /tmp/generated-inventory.ini
    
    - name: Display generated inventory
      command: cat /tmp/generated-inventory.ini
      register: inventory
      changed_when: false
    
    - name: Show inventory
      debug:
        var: inventory.stdout_lines
EOF
```

### Step 10: Test Complex Data

```bash
ansible-playbook complex-data.yml
cat /tmp/generated-inventory.ini
```

## Expected Results

### Facts Display

```
TASK [Display system information] ********************************************
ok: [localhost] => {
    "msg": "Hostname: myhost\nFQDN: myhost.local\nOS: Ubuntu 22.04\n..."
}
```

### Filter Output

```
TASK [Select attribute filter] ***********************************************
ok: [localhost] => {
    "msg": "Active users: ['alice', 'bob', 'david']"
}
```

## Common Ansible Filters

| Filter | Example | Description |
|--------|---------|-------------|
| `default` | `{{ var \| default('N/A') }}` | Provide default value |
| `map` | `{{ list \| map(attribute='key') }}` | Extract attribute from list |
| `select` | `{{ list \| select('defined') }}` | Filter items |
| `selectattr` | `{{ list \| selectattr('key', 'eq', 'value') }}` | Filter by attribute |
| `rejectattr` | `{{ list \| rejectattr('key', 'eq', 'value') }}` | Reject by attribute |
| `unique` | `{{ list \| unique }}` | Remove duplicates |
| `combine` | `{{ dict1 \| combine(dict2) }}` | Merge dictionaries |
| `to_json` | `{{ var \| to_json }}` | Convert to JSON |
| `to_yaml` | `{{ var \| to_yaml }}` | Convert to YAML |
| `regex_replace` | `{{ str \| regex_replace('old', 'new') }}` | Regex substitution |

## Troubleshooting

### Problem: Fact not available

**Solution:** Check gather_facts setting or re-gather facts
```yaml
- name: Gather facts
  setup:
```

### Problem: Custom fact not loading

**Solution:** Check file location and permissions
```bash
ls -la /etc/ansible/facts.d/
```

### Problem: Filter returns empty

**Solution:** Debug the data structure first
```yaml
- debug:
    var: my_variable
```

## Additional Exercises

1. Create custom facts for application monitoring
2. Build a comprehensive system report using facts
3. Filter and group servers by attributes
4. Create dynamic inventories from facts
5. Practice with all major filter types

## What You Learned

✅ Gathering and using system facts  
✅ Creating custom facts  
✅ Using set_fact for custom variables  
✅ Advanced filtering with selectattr and rejectattr  
✅ Mapping and transforming data  
✅ Dictionary and list manipulation  
✅ JSON and YAML conversions  
✅ Complex data structure handling  

## Next Steps

Move on to [Exercise 15: Vault Secrets](../exercise-15/README.md) to learn about encrypting sensitive data.
