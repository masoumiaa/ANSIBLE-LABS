# Exercise 19: Dynamic Inventory

## Learning Objectives

- Create dynamic inventory scripts
- Generate inventory from external sources
- Use inventory plugins
- Manage dynamic host lists

## Prerequisites

- Completed Exercise 18
- Understanding of inventory basics
- Basic scripting knowledge

## Steps

### Step 1: Create a Simple Dynamic Inventory Script

Create a Python script that returns inventory:

```bash
cat > /tmp/simple-inventory.py << 'EOF'
#!/usr/bin/env python3
import json

inventory = {
    "webservers": {
        "hosts": ["web1", "web2", "web3"],
        "vars": {
            "http_port": 80,
            "ansible_connection": "local"
        }
    },
    "databases": {
        "hosts": ["db1", "db2"],
        "vars": {
            "db_port": 5432,
            "ansible_connection": "local"
        }
    },
    "production": {
        "children": ["webservers", "databases"],
        "vars": {
            "environment": "production"
        }
    },
    "_meta": {
        "hostvars": {
            "web1": {"ansible_host": "localhost", "server_id": 1},
            "web2": {"ansible_host": "localhost", "server_id": 2},
            "web3": {"ansible_host": "localhost", "server_id": 3},
            "db1": {"ansible_host": "localhost", "server_id": 10},
            "db2": {"ansible_host": "localhost", "server_id": 11}
        }
    }
}

print(json.dumps(inventory, indent=2))
EOF

chmod +x /tmp/simple-inventory.py
```

### Step 2: Test Dynamic Inventory Script

```bash
# Test the script
/tmp/simple-inventory.py

# Use with ansible
ansible-inventory -i /tmp/simple-inventory.py --list

# Use in a playbook
cat > test-dynamic-inventory.yml << 'EOF'
---
- name: Test Dynamic Inventory
  hosts: webservers
  gather_facts: no
  
  tasks:
    - name: Display host information
      debug:
        msg: |
          Host: {{ inventory_hostname }}
          Server ID: {{ server_id }}
          Port: {{ http_port }}
          Environment: {{ environment }}
EOF

ansible-playbook -i /tmp/simple-inventory.py test-dynamic-inventory.yml --limit web1
```

### Step 3: Create Dynamic Inventory from JSON File

Create a script that reads from a JSON file:

```bash
# Create hosts data file
cat > /tmp/hosts-data.json << 'EOF'
{
  "servers": [
    {
      "name": "app-server-1",
      "role": "application",
      "environment": "production",
      "ip": "192.168.1.10",
      "cpu": 4,
      "memory": 8192
    },
    {
      "name": "app-server-2",
      "role": "application",
      "environment": "production",
      "ip": "192.168.1.11",
      "cpu": 4,
      "memory": 8192
    },
    {
      "name": "db-server-1",
      "role": "database",
      "environment": "production",
      "ip": "192.168.1.20",
      "cpu": 8,
      "memory": 16384
    },
    {
      "name": "cache-server-1",
      "role": "cache",
      "environment": "production",
      "ip": "192.168.1.30",
      "cpu": 2,
      "memory": 4096
    }
  ]
}
EOF

# Create inventory generator script
cat > /tmp/generate-inventory.py << 'EOF'
#!/usr/bin/env python3
import json
import sys

def generate_inventory():
    # Read hosts data
    with open('/tmp/hosts-data.json', 'r') as f:
        data = json.load(f)
    
    inventory = {
        "_meta": {
            "hostvars": {}
        }
    }
    
    # Group servers by role and environment
    for server in data['servers']:
        role = server['role']
        env = server['environment']
        name = server['name']
        
        # Create role group
        if role not in inventory:
            inventory[role] = {
                "hosts": [],
                "vars": {}
            }
        inventory[role]["hosts"].append(name)
        
        # Create environment group
        if env not in inventory:
            inventory[env] = {
                "hosts": [],
                "vars": {}
            }
        inventory[env]["hosts"].append(name)
        
        # Add host variables
        inventory["_meta"]["hostvars"][name] = {
            "ansible_host": "localhost",  # For demo purposes
            "ansible_connection": "local",
            "real_ip": server['ip'],
            "cpu_count": server['cpu'],
            "memory_mb": server['memory'],
            "server_role": role,
            "server_env": env
        }
    
    return inventory

if __name__ == "__main__":
    inventory = generate_inventory()
    print(json.dumps(inventory, indent=2))
EOF

chmod +x /tmp/generate-inventory.py
```

### Step 4: Test JSON-based Inventory

```bash
# Test the generator
/tmp/generate-inventory.py

# List inventory
ansible-inventory -i /tmp/generate-inventory.py --list

# Show inventory graph
ansible-inventory -i /tmp/generate-inventory.py --graph

# Use in playbook
cat > use-generated-inventory.yml << 'EOF'
---
- name: Use Generated Inventory
  hosts: all
  gather_facts: no
  
  tasks:
    - name: Display server information
      debug:
        msg: |
          Server: {{ inventory_hostname }}
          Role: {{ server_role }}
          Environment: {{ server_env }}
          CPU: {{ cpu_count }} cores
          Memory: {{ memory_mb }} MB
          Real IP: {{ real_ip }}
EOF

ansible-playbook -i /tmp/generate-inventory.py use-generated-inventory.yml --limit application
```

### Step 5: Create Dynamic Inventory with Filtering

Create a script with filtering capabilities:

```bash
cat > /tmp/filtered-inventory.py << 'EOF'
#!/usr/bin/env python3
import json
import sys
import os

def get_servers():
    """Simulate fetching servers from a source"""
    return [
        {"name": "web-prod-1", "env": "prod", "role": "web", "region": "us-east", "active": True},
        {"name": "web-prod-2", "env": "prod", "role": "web", "region": "us-east", "active": True},
        {"name": "web-staging-1", "env": "staging", "role": "web", "region": "us-west", "active": True},
        {"name": "api-prod-1", "env": "prod", "role": "api", "region": "us-east", "active": True},
        {"name": "api-prod-2", "env": "prod", "role": "api", "region": "us-east", "active": False},
        {"name": "db-prod-1", "env": "prod", "role": "database", "region": "us-east", "active": True},
        {"name": "db-staging-1", "env": "staging", "role": "database", "region": "us-west", "active": True},
    ]

def generate_inventory():
    servers = get_servers()
    
    # Filter based on environment variable
    filter_env = os.environ.get('INVENTORY_ENV', None)
    filter_active = os.environ.get('INVENTORY_ACTIVE_ONLY', 'false').lower() == 'true'
    
    if filter_env:
        servers = [s for s in servers if s['env'] == filter_env]
    
    if filter_active:
        servers = [s for s in servers if s['active']]
    
    inventory = {
        "_meta": {"hostvars": {}},
        "all": {"hosts": [], "vars": {}}
    }
    
    for server in servers:
        name = server['name']
        
        # Add to all
        inventory["all"]["hosts"].append(name)
        
        # Create groups
        for key in ['env', 'role', 'region']:
            group = server[key]
            if group not in inventory:
                inventory[group] = {"hosts": [], "vars": {}}
            inventory[group]["hosts"].append(name)
        
        # Host vars
        inventory["_meta"]["hostvars"][name] = {
            "ansible_host": "localhost",
            "ansible_connection": "local",
            "environment": server['env'],
            "role": server['role'],
            "region": server['region'],
            "active": server['active']
        }
    
    return inventory

if __name__ == "__main__":
    inventory = generate_inventory()
    print(json.dumps(inventory, indent=2))
EOF

chmod +x /tmp/filtered-inventory.py
```

### Step 6: Test Filtered Inventory

```bash
# All servers
/tmp/filtered-inventory.py

# Production only
INVENTORY_ENV=prod /tmp/filtered-inventory.py

# Active servers only
INVENTORY_ACTIVE_ONLY=true /tmp/filtered-inventory.py

# Production and active only
INVENTORY_ENV=prod INVENTORY_ACTIVE_ONLY=true /tmp/filtered-inventory.py

# Use with ansible
ansible-inventory -i /tmp/filtered-inventory.py --graph

# Use in playbook
cat > filtered-playbook.yml << 'EOF'
---
- name: Filtered Inventory Playbook
  hosts: prod
  gather_facts: no
  
  tasks:
    - name: Display active production servers
      debug:
        msg: "{{ inventory_hostname }} - {{ role }} in {{ region }}"
      when: active
EOF

INVENTORY_ENV=prod ansible-playbook -i /tmp/filtered-inventory.py filtered-playbook.yml
```

### Step 7: Create Inventory from CSV

```bash
# Create CSV file
cat > /tmp/servers.csv << 'EOF'
hostname,ip,role,environment,datacenter
srv-web-01,10.0.1.10,webserver,production,dc1
srv-web-02,10.0.1.11,webserver,production,dc1
srv-api-01,10.0.2.10,api,production,dc1
srv-db-01,10.0.3.10,database,production,dc2
srv-cache-01,10.0.4.10,cache,production,dc1
EOF

# Create CSV to inventory converter
cat > /tmp/csv-to-inventory.py << 'EOF'
#!/usr/bin/env python3
import json
import csv

def csv_to_inventory(csv_file):
    inventory = {
        "_meta": {"hostvars": {}},
        "all": {"hosts": []}
    }
    
    with open(csv_file, 'r') as f:
        reader = csv.DictReader(f)
        for row in reader:
            hostname = row['hostname']
            
            # Add to all hosts
            inventory["all"]["hosts"].append(hostname)
            
            # Create groups by role, environment, datacenter
            for group_key in ['role', 'environment', 'datacenter']:
                group = row[group_key]
                if group not in inventory:
                    inventory[group] = {"hosts": []}
                inventory[group]["hosts"].append(hostname)
            
            # Host vars
            inventory["_meta"]["hostvars"][hostname] = {
                "ansible_host": "localhost",  # Demo
                "ansible_connection": "local",
                "real_ip": row['ip'],
                "server_role": row['role'],
                "environment": row['environment'],
                "datacenter": row['datacenter']
            }
    
    return inventory

if __name__ == "__main__":
    inventory = csv_to_inventory('/tmp/servers.csv')
    print(json.dumps(inventory, indent=2))
EOF

chmod +x /tmp/csv-to-inventory.py
```

### Step 8: Test CSV Inventory

```bash
# Test CSV inventory
/tmp/csv-to-inventory.py

# List servers by role
ansible-inventory -i /tmp/csv-to-inventory.py --graph

# Use in playbook
cat > csv-playbook.yml << 'EOF'
---
- name: CSV Inventory Playbook
  hosts: webserver
  gather_facts: no
  
  tasks:
    - name: Display webserver information
      debug:
        msg: |
          Server: {{ inventory_hostname }}
          IP: {{ real_ip }}
          Environment: {{ environment }}
          Datacenter: {{ datacenter }}
EOF

ansible-playbook -i /tmp/csv-to-inventory.py csv-playbook.yml
```

### Step 9: Dynamic Inventory with Caching

Create a script with caching support:

```bash
cat > /tmp/cached-inventory.py << 'EOF'
#!/usr/bin/env python3
import json
import os
import time

CACHE_FILE = '/tmp/inventory-cache.json'
CACHE_TTL = 300  # 5 minutes

def fetch_inventory():
    """Simulate expensive API call"""
    print("Fetching inventory from API...", file=sys.stderr)
    time.sleep(1)  # Simulate API delay
    
    return {
        "webservers": {
            "hosts": ["web1", "web2"],
            "vars": {"http_port": 80}
        },
        "_meta": {
            "hostvars": {
                "web1": {"ansible_host": "localhost", "ansible_connection": "local"},
                "web2": {"ansible_host": "localhost", "ansible_connection": "local"}
            }
        }
    }

def get_cached_inventory():
    if os.path.exists(CACHE_FILE):
        mtime = os.path.getmtime(CACHE_FILE)
        if time.time() - mtime < CACHE_TTL:
            with open(CACHE_FILE, 'r') as f:
                print("Using cached inventory", file=sys.stderr)
                return json.load(f)
    
    inventory = fetch_inventory()
    
    with open(CACHE_FILE, 'w') as f:
        json.dump(inventory, f)
    
    return inventory

if __name__ == "__main__":
    import sys
    inventory = get_cached_inventory()
    print(json.dumps(inventory, indent=2))
EOF

chmod +x /tmp/cached-inventory.py
```

### Step 10: Create Inventory Construction Script

```bash
cat > /tmp/construct-inventory.py << 'EOF'
#!/usr/bin/env python3
"""
Complete dynamic inventory example with various features
"""
import json
import sys
import argparse

class DynamicInventory:
    def __init__(self):
        self.inventory = {
            "_meta": {"hostvars": {}},
            "all": {"vars": {
                "ansible_connection": "local",
                "deployment_date": "2026-01-28"
            }}
        }
    
    def add_host(self, hostname, groups, hostvars=None):
        """Add a host to inventory"""
        if hostvars:
            self.inventory["_meta"]["hostvars"][hostname] = hostvars
        
        for group in groups:
            if group not in self.inventory:
                self.inventory[group] = {"hosts": []}
            self.inventory[group]["hosts"].append(hostname)
    
    def add_group_vars(self, group, vars):
        """Add variables to a group"""
        if group not in self.inventory:
            self.inventory[group] = {"hosts": []}
        if "vars" not in self.inventory[group]:
            self.inventory[group]["vars"] = {}
        self.inventory[group]["vars"].update(vars)
    
    def add_child_group(self, parent, child):
        """Add a child group to a parent"""
        if parent not in self.inventory:
            self.inventory[parent] = {}
        if "children" not in self.inventory[parent]:
            self.inventory[parent]["children"] = []
        self.inventory[parent]["children"].append(child)
    
    def get_inventory(self):
        return self.inventory

def build_infrastructure():
    inv = DynamicInventory()
    
    # Web servers
    for i in range(1, 4):
        inv.add_host(
            f"web-{i}",
            ["webservers", "frontend", "production"],
            {
                "ansible_host": "localhost",
                "server_id": i,
                "port": 8080 + i
            }
        )
    
    # API servers
    for i in range(1, 3):
        inv.add_host(
            f"api-{i}",
            ["api_servers", "backend", "production"],
            {
                "ansible_host": "localhost",
                "server_id": 10 + i,
                "port": 3000 + i
            }
        )
    
    # Database servers
    for i in range(1, 3):
        inv.add_host(
            f"db-{i}",
            ["databases", "backend", "production"],
            {
                "ansible_host": "localhost",
                "server_id": 20 + i,
                "port": 5432,
                "is_primary": i == 1
            }
        )
    
    # Add group variables
    inv.add_group_vars("webservers", {"max_connections": 100, "enable_ssl": True})
    inv.add_group_vars("api_servers", {"max_workers": 4, "timeout": 30})
    inv.add_group_vars("databases", {"max_connections": 200, "backup_enabled": True})
    
    # Add group hierarchy
    inv.add_child_group("production", "frontend")
    inv.add_child_group("production", "backend")
    
    return inv.get_inventory()

if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument('--list', action='store_true')
    parser.add_argument('--host', action='store')
    args = parser.parse_args()
    
    if args.list:
        inventory = build_infrastructure()
        print(json.dumps(inventory, indent=2))
    elif args.host:
        # Return specific host vars (optional optimization)
        print(json.dumps({}))
    else:
        print(json.dumps({}))
EOF

chmod +x /tmp/construct-inventory.py
```

### Step 11: Test Complete Inventory

```bash
# List full inventory
/tmp/construct-inventory.py --list

# View as graph
ansible-inventory -i /tmp/construct-inventory.py --graph

# Use in comprehensive playbook
cat > complete-dynamic-playbook.yml << 'EOF'
---
- name: Infrastructure Deployment
  hosts: production
  gather_facts: no
  
  tasks:
    - name: Display infrastructure information
      debug:
        msg: |
          Host: {{ inventory_hostname }}
          Server ID: {{ server_id }}
          Groups: {{ group_names }}

- name: Configure Web Servers
  hosts: webservers
  gather_facts: no
  
  tasks:
    - name: Configure webserver
      debug:
        msg: |
          Configuring {{ inventory_hostname }}
          Port: {{ port }}
          Max Connections: {{ max_connections }}
          SSL: {{ enable_ssl }}

- name: Configure Databases
  hosts: databases
  gather_facts: no
  
  tasks:
    - name: Configure database
      debug:
        msg: |
          Configuring {{ inventory_hostname }}
          {{ 'PRIMARY' if is_primary else 'REPLICA' }}
          Backup: {{ backup_enabled }}
EOF

ansible-playbook -i /tmp/construct-inventory.py complete-dynamic-playbook.yml
```

### Step 12: Inventory Script Template

Create a template for custom inventory scripts:

```bash
cat > /tmp/inventory-template.py << 'EOF'
#!/usr/bin/env python3
"""
Template for creating custom dynamic inventory scripts
"""
import json
import sys

def get_inventory():
    """
    Main function to generate inventory
    Modify this function to pull data from your source:
    - Database
    - Cloud API (AWS, Azure, GCP)
    - CMDB
    - Custom API
    - Configuration files
    """
    
    inventory = {
        # Meta section for host variables
        "_meta": {
            "hostvars": {
                # Example host variables
                "example-host": {
                    "ansible_host": "192.168.1.10",
                    "ansible_user": "admin",
                    "custom_var": "value"
                }
            }
        },
        
        # Example group
        "example_group": {
            "hosts": ["example-host"],
            "vars": {
                "group_var": "group_value"
            }
        },
        
        # Group with children
        "parent_group": {
            "children": ["example_group"],
            "vars": {
                "parent_var": "parent_value"
            }
        }
    }
    
    return inventory

if __name__ == "__main__":
    # Generate and print inventory
    inventory = get_inventory()
    print(json.dumps(inventory, indent=2))
EOF

chmod +x /tmp/inventory-template.py
```

## Expected Results

### Dynamic Inventory Output

```json
{
  "webservers": {
    "hosts": ["web1", "web2", "web3"],
    "vars": {
      "http_port": 80
    }
  },
  "_meta": {
    "hostvars": {
      "web1": {
        "ansible_host": "localhost",
        "server_id": 1
      }
    }
  }
}
```

### Inventory Graph

```
@all:
  |--@production:
  |  |--@databases:
  |  |  |--db1
  |  |  |--db2
  |  |--@webservers:
  |  |  |--web1
  |  |  |--web2
```

## Dynamic Inventory Script Requirements

### Minimum Requirements
```python
{
  "_meta": {
    "hostvars": {}
  },
  "group_name": {
    "hosts": ["host1", "host2"]
  }
}
```

### Command Line Arguments
- `--list`: Return full inventory
- `--host <hostname>`: Return vars for specific host

## Troubleshooting

### Problem: Script not executable

**Solution:** Make script executable
```bash
chmod +x inventory-script.py
```

### Problem: JSON parsing error

**Solution:** Validate JSON output
```bash
./inventory-script.py | python3 -m json.tool
```

### Problem: Slow inventory generation

**Solution:** Implement caching and use `_meta` section

## Additional Exercises

1. Create inventory from AWS/Azure API
2. Build inventory from database queries
3. Implement inventory caching mechanism
4. Create inventory with complex group hierarchies
5. Build inventory from multiple sources

## What You Learned

✅ Creating dynamic inventory scripts  
✅ JSON format for dynamic inventories  
✅ Filtering and grouping hosts  
✅ Host and group variables  
✅ Inventory caching  
✅ Reading from external sources  
✅ Complex inventory structures  
✅ Performance optimization  

## Next Steps

Move on to [Exercise 20: Full Web Server Deployment](../exercise-20/README.md) for a complete practical project.
