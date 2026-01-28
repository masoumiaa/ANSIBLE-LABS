# Exercise 10: Package Management

## Learning Objectives

- Install and remove packages
- Update package cache
- Work with different package managers
- Manage package versions

## Prerequisites

- Completed Exercise 09
- Understanding of Linux package management
- May require sudo/root access

## Steps

### Step 1: Basic Package Operations (Debian/Ubuntu)

Create a playbook for apt package management:

```bash
cat > apt-packages.yml << 'EOF'
---
- name: Package Management with APT
  hosts: localhost
  connection: local
  become: yes
  
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600  # Cache valid for 1 hour
      when: ansible_os_family == "Debian"
    
    - name: Install single package
      apt:
        name: curl
        state: present
      when: ansible_os_family == "Debian"
    
    - name: Install multiple packages
      apt:
        name:
          - wget
          - git
          - vim
        state: present
      when: ansible_os_family == "Debian"
    
    - name: Install specific version
      apt:
        name: curl=7.68.0-1ubuntu2.22
        state: present
      when: ansible_os_family == "Debian"
      ignore_errors: yes  # Version might not exist
    
    - name: Ensure package is latest version
      apt:
        name: curl
        state: latest
      when: ansible_os_family == "Debian"
    
    - name: Check if package is installed
      command: dpkg -l curl
      register: curl_check
      ignore_errors: yes
      changed_when: false
      when: ansible_os_family == "Debian"
    
    - name: Display package status
      debug:
        msg: "Curl is installed"
      when: 
        - ansible_os_family == "Debian"
        - curl_check.rc == 0
EOF
```

### Step 2: Test APT Package Management

```bash
# Run with sudo (if on Debian/Ubuntu)
ansible-playbook apt-packages.yml

# Or run in check mode to see what would change
ansible-playbook apt-packages.yml --check
```

### Step 3: Package Management with YUM/DNF (RedHat/CentOS)

Create a playbook for yum/dnf:

```bash
cat > yum-packages.yml << 'EOF'
---
- name: Package Management with YUM/DNF
  hosts: localhost
  connection: local
  become: yes
  
  tasks:
    - name: Install package with yum
      yum:
        name: curl
        state: present
      when: ansible_os_family == "RedHat" and ansible_distribution_major_version|int < 8
    
    - name: Install package with dnf (Fedora/RHEL 8+)
      dnf:
        name: curl
        state: present
      when: ansible_os_family == "RedHat" and ansible_distribution_major_version|int >= 8
    
    - name: Install multiple packages
      yum:
        name:
          - wget
          - git
          - vim
        state: present
      when: ansible_os_family == "RedHat"
    
    - name: Update all packages
      yum:
        name: '*'
        state: latest
      when: ansible_os_family == "RedHat"
      tags: never  # Skip by default, run with --tags update
    
    - name: Remove a package
      yum:
        name: nano
        state: absent
      when: ansible_os_family == "RedHat"
EOF
```

### Step 4: Generic Package Module

Create a cross-platform package playbook:

```bash
cat > generic-packages.yml << 'EOF'
---
- name: Cross-Platform Package Management
  hosts: localhost
  connection: local
  gather_facts: yes
  
  tasks:
    - name: Display OS information
      debug:
        msg: "OS: {{ ansible_os_family }}, Distribution: {{ ansible_distribution }}"
    
    - name: Install package (generic module)
      package:
        name: "{{ item }}"
        state: present
      loop:
        - curl
        - wget
        - git
      become: yes
      ignore_errors: yes  # Some packages might not exist
    
    - name: Check which packages are installed
      command: which {{ item }}
      loop:
        - curl
        - wget
        - git
      register: package_check
      ignore_errors: yes
      changed_when: false
    
    - name: Display installed packages
      debug:
        msg: "{{ item.item }} is installed at: {{ item.stdout }}"
      loop: "{{ package_check.results }}"
      when: item.rc == 0
EOF
```

### Step 5: Test Generic Package Module

```bash
ansible-playbook generic-packages.yml
```

### Step 6: Package Installation with Loops

Create a playbook that installs packages with specific configurations:

```bash
cat > package-loops.yml << 'EOF'
---
- name: Install Packages with Configuration
  hosts: localhost
  connection: local
  vars:
    packages_to_install:
      - name: curl
        state: present
      - name: wget
        state: present
      - name: git
        state: latest
      - name: tree
        state: present
  
  tasks:
    - name: Display packages to be managed
      debug:
        msg: "Package: {{ item.name }}, State: {{ item.state }}"
      loop: "{{ packages_to_install }}"
    
    - name: Manage packages
      package:
        name: "{{ item.name }}"
        state: "{{ item.state }}"
      loop: "{{ packages_to_install }}"
      become: yes
      register: package_result
      ignore_errors: yes
    
    - name: Display installation results
      debug:
        msg: "{{ item.item.name }}: {{ 'Installed' if item.changed else 'Already present' }}"
      loop: "{{ package_result.results }}"
      when: not item.failed
EOF
```

### Step 7: Run Package Loop Playbook

```bash
ansible-playbook package-loops.yml
```

### Step 8: Package Facts and Verification

Create a playbook to gather package information:

```bash
cat > package-facts.yml << 'EOF'
---
- name: Package Facts and Verification
  hosts: localhost
  connection: local
  gather_facts: yes
  
  tasks:
    - name: Gather package facts
      package_facts:
        manager: auto
    
    - name: Display all installed packages (first 10)
      debug:
        msg: "{{ ansible_facts.packages.keys() | list | first(10) }}"
    
    - name: Check if specific package is installed
      debug:
        msg: "curl is installed, version: {{ ansible_facts.packages['curl'][0].version }}"
      when: "'curl' in ansible_facts.packages"
    
    - name: Check if package is NOT installed
      debug:
        msg: "nginx is not installed"
      when: "'nginx' not in ansible_facts.packages"
    
    - name: Verify multiple packages
      debug:
        msg: "{{ item }} is {{ 'installed' if item in ansible_facts.packages else 'NOT installed' }}"
      loop:
        - curl
        - wget
        - git
        - nginx
        - apache2
    
    - name: Count total installed packages
      debug:
        msg: "Total packages installed: {{ ansible_facts.packages | length }}"
EOF
```

### Step 9: Test Package Facts

```bash
ansible-playbook package-facts.yml
```

### Step 10: Complete Package Management Example

Create a comprehensive package management playbook:

```bash
cat > package-management.yml << 'EOF'
---
- name: Complete Package Management
  hosts: localhost
  connection: local
  become: yes
  vars:
    required_packages:
      - curl
      - wget
      - git
    optional_packages:
      - vim
      - nano
    packages_to_remove:
      - telnet
  
  tasks:
    - name: Update package cache
      package_facts:
        manager: auto
    
    - name: Display current OS
      debug:
        msg: "Managing packages on {{ ansible_distribution }} {{ ansible_distribution_version }}"
    
    - name: Ensure required packages are installed
      package:
        name: "{{ required_packages }}"
        state: present
      register: required_install
    
    - name: Report required packages installation
      debug:
        msg: "Required packages installation completed"
      when: required_install.changed
    
    - name: Ensure optional packages are installed
      package:
        name: "{{ optional_packages }}"
        state: present
      ignore_errors: yes
      register: optional_install
    
    - name: Remove unwanted packages
      package:
        name: "{{ packages_to_remove }}"
        state: absent
      ignore_errors: yes
    
    - name: Verify required packages
      command: which {{ item }}
      loop: "{{ required_packages }}"
      register: verify_packages
      changed_when: false
      failed_when: false
    
    - name: Create installation report
      copy:
        content: |
          Package Management Report
          =========================
          Date: {{ ansible_date_time.iso8601 }}
          Host: {{ ansible_hostname }}
          OS: {{ ansible_distribution }} {{ ansible_distribution_version }}
          
          Required Packages:
          {% for pkg in required_packages %}
          - {{ pkg }}
          {% endfor %}
          
          Status: Completed
        dest: /tmp/package-report.txt
    
    - name: Display report location
      debug:
        msg: "Package report saved to /tmp/package-report.txt"
EOF
```

### Step 11: Run Complete Package Management

```bash
ansible-playbook package-management.yml

# View the report
cat /tmp/package-report.txt
```

## Expected Results

### Package Installation Output

```
TASK [Install single package] ************************************************
changed: [localhost]

TASK [Install multiple packages] *********************************************
changed: [localhost]
```

### Package Already Installed

```
TASK [Install single package] ************************************************
ok: [localhost]
```

### Package Facts Output

```json
{
    "ansible_facts": {
        "packages": {
            "curl": [
                {
                    "name": "curl",
                    "version": "7.68.0-1ubuntu2.22"
                }
            ]
        }
    }
}
```

## Package Module Reference

| Module | OS Family | Description |
|--------|-----------|-------------|
| `apt` | Debian/Ubuntu | Debian package manager |
| `yum` | RHEL/CentOS 7 | RedHat package manager |
| `dnf` | RHEL/CentOS 8+, Fedora | Modern RedHat package manager |
| `package` | All | Generic cross-platform module |
| `package_facts` | All | Gather package information |

## Common Package States

- `present` - Ensure package is installed
- `absent` - Ensure package is not installed
- `latest` - Ensure package is at latest version

## Troubleshooting

### Problem: "Permission denied"

**Solution:** Use `become: yes`
```yaml
- name: Install package
  package:
    name: curl
    state: present
  become: yes
```

### Problem: Package not found

**Solution:** Update cache first or check package name
```yaml
- name: Update cache
  apt:
    update_cache: yes
```

### Problem: Specific version not available

**Solution:** Use `ignore_errors` or check available versions
```bash
apt-cache policy package_name
```

## Additional Exercises

1. Create a playbook that installs a LAMP stack
2. Install packages conditionally based on OS family
3. Create a package removal playbook
4. Build a playbook that updates all system packages
5. Write a role that manages development tools

## What You Learned

✅ Installing packages with apt, yum, and dnf  
✅ Using the generic package module  
✅ Installing multiple packages with loops  
✅ Gathering package facts  
✅ Checking if packages are installed  
✅ Managing package versions  
✅ Cross-platform package management  
✅ Package verification and reporting  

## Next Steps

Move on to [Exercise 11: Using Variables](../exercise-11/README.md) to learn about variable management in Ansible.
