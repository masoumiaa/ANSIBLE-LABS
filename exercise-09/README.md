# Exercise 09: File Management

## Learning Objectives

- Manage files and directories
- Copy and template files
- Modify file permissions
- Use file content manipulation modules

## Prerequisites

- Completed Exercise 08
- Understanding of file permissions

## Steps

### Step 1: Basic File Operations

Create a playbook for basic file management:

```bash
cat > file-operations.yml << 'EOF'
---
- name: Basic File Operations
  hosts: localhost
  connection: local
  
  tasks:
    - name: Create a directory
      file:
        path: /tmp/fileops
        state: directory
        mode: '0755'
    
    - name: Create an empty file
      file:
        path: /tmp/fileops/empty.txt
        state: touch
        mode: '0644'
    
    - name: Create a file with specific owner (if possible)
      file:
        path: /tmp/fileops/owned.txt
        state: touch
        mode: '0644'
    
    - name: Create multiple nested directories
      file:
        path: /tmp/fileops/level1/level2/level3
        state: directory
        mode: '0755'
    
    - name: Create a symbolic link
      file:
        src: /tmp/fileops/empty.txt
        dest: /tmp/fileops/link-to-empty.txt
        state: link
    
    - name: Get file information
      stat:
        path: /tmp/fileops/empty.txt
      register: file_info
    
    - name: Display file information
      debug:
        msg: |
          File exists: {{ file_info.stat.exists }}
          File size: {{ file_info.stat.size }} bytes
          File mode: {{ file_info.stat.mode }}
          Is directory: {{ file_info.stat.isdir }}
EOF
```

### Step 2: Run File Operations

```bash
ansible-playbook file-operations.yml

# Verify results
ls -la /tmp/fileops/
```

### Step 3: Copy Module Examples

Create a playbook using the copy module:

```bash
cat > copy-examples.yml << 'EOF'
---
- name: Copy Module Examples
  hosts: localhost
  connection: local
  
  tasks:
    - name: Create working directory
      file:
        path: /tmp/copy-test
        state: directory
    
    - name: Copy inline content to file
      copy:
        content: |
          This is a test file.
          Created by Ansible copy module.
          Line 3 of content.
        dest: /tmp/copy-test/inline.txt
        mode: '0644'
    
    - name: Copy with backup
      copy:
        content: "Version 1\n"
        dest: /tmp/copy-test/versioned.txt
        backup: yes
    
    - name: Update file (creates backup)
      copy:
        content: "Version 2\n"
        dest: /tmp/copy-test/versioned.txt
        backup: yes
    
    - name: Copy only if different
      copy:
        content: "Same content\n"
        dest: /tmp/copy-test/idempotent.txt
    
    - name: Run again - no change
      copy:
        content: "Same content\n"
        dest: /tmp/copy-test/idempotent.txt
      register: copy_result
    
    - name: Show if file changed
      debug:
        msg: "File changed: {{ copy_result.changed }}"
    
    - name: Copy with validation (creates then validates)
      copy:
        content: |
          key1=value1
          key2=value2
        dest: /tmp/copy-test/config.ini
        validate: 'test -f %s'
EOF
```

### Step 4: Test Copy Module

```bash
ansible-playbook copy-examples.yml

# Check backup files
ls -la /tmp/copy-test/
```

### Step 5: File Content Manipulation

Create a playbook for modifying file contents:

```bash
cat > file-content.yml << 'EOF'
---
- name: File Content Manipulation
  hosts: localhost
  connection: local
  
  tasks:
    - name: Create working directory
      file:
        path: /tmp/content-test
        state: directory
    
    - name: Create initial configuration file
      copy:
        content: |
          # Application Configuration
          debug=false
          port=8080
          host=localhost
        dest: /tmp/content-test/app.conf
    
    - name: Add a line to file
      lineinfile:
        path: /tmp/content-test/app.conf
        line: "timeout=30"
        create: yes
    
    - name: Ensure a line exists (idempotent)
      lineinfile:
        path: /tmp/content-test/app.conf
        line: "# Auto-generated configuration"
        insertbefore: BOF  # Beginning of file
    
    - name: Replace a line matching pattern
      lineinfile:
        path: /tmp/content-test/app.conf
        regexp: '^debug='
        line: "debug=true"
    
    - name: Remove a line
      lineinfile:
        path: /tmp/content-test/app.conf
        regexp: '^host='
        state: absent
    
    - name: Read and display file
      slurp:
        src: /tmp/content-test/app.conf
      register: config_content
    
    - name: Show file content
      debug:
        msg: "{{ config_content.content | b64decode }}"
    
    - name: Use blockinfile for multiline content
      blockinfile:
        path: /tmp/content-test/app.conf
        block: |
          # Database Configuration
          db_host=localhost
          db_port=5432
          db_name=myapp
        marker: "# {mark} ANSIBLE MANAGED BLOCK"
EOF
```

### Step 6: Test Content Manipulation

```bash
ansible-playbook file-content.yml

# View the modified file
cat /tmp/content-test/app.conf
```

### Step 7: File Permissions Management

Create a playbook for permission management:

```bash
cat > permissions.yml << 'EOF'
---
- name: File Permissions Management
  hosts: localhost
  connection: local
  
  tasks:
    - name: Create test directory
      file:
        path: /tmp/perms-test
        state: directory
    
    - name: Create file with specific permissions
      file:
        path: /tmp/perms-test/file1.txt
        state: touch
        mode: '0644'  # rw-r--r--
    
    - name: Create file with different permissions
      file:
        path: /tmp/perms-test/file2.txt
        state: touch
        mode: '0600'  # rw-------
    
    - name: Create executable script
      copy:
        content: |
          #!/bin/bash
          echo "Hello from script"
        dest: /tmp/perms-test/script.sh
        mode: '0755'  # rwxr-xr-x
    
    - name: Change permissions on existing file
      file:
        path: /tmp/perms-test/file1.txt
        mode: '0400'  # r--------
    
    - name: Set permissions with symbolic mode
      file:
        path: /tmp/perms-test/file2.txt
        mode: 'u+x,g+r,o-rwx'
    
    - name: Get permission information
      stat:
        path: /tmp/perms-test/script.sh
      register: script_stat
    
    - name: Display permission info
      debug:
        msg: |
          File: script.sh
          Mode: {{ script_stat.stat.mode }}
          Executable: {{ script_stat.stat.executable }}
          Readable: {{ script_stat.stat.readable }}
          Writable: {{ script_stat.stat.writeable }}
    
    - name: List all files with permissions
      command: ls -la /tmp/perms-test/
      register: file_list
    
    - name: Show file listing
      debug:
        var: file_list.stdout_lines
EOF
```

### Step 8: Test Permissions

```bash
ansible-playbook permissions.yml

# Verify permissions
ls -la /tmp/perms-test/
```

### Step 9: Advanced File Operations

Create a playbook with advanced file operations:

```bash
cat > advanced-files.yml << 'EOF'
---
- name: Advanced File Operations
  hosts: localhost
  connection: local
  
  tasks:
    - name: Create working directory
      file:
        path: /tmp/advanced-files
        state: directory
    
    - name: Find files matching pattern
      find:
        paths: /tmp
        patterns: "*.txt"
        recurse: no
      register: found_files
    
    - name: Display found files
      debug:
        msg: "Found {{ found_files.matched }} text files"
    
    - name: Create test files for demonstration
      copy:
        content: "Test file {{ item }}\n"
        dest: "/tmp/advanced-files/test{{ item }}.txt"
      loop: [1, 2, 3, 4, 5]
    
    - name: Find recently created files
      find:
        paths: /tmp/advanced-files
        patterns: "*.txt"
        age: "-1m"  # Modified in last 1 minute
      register: recent_files
    
    - name: Show recent files
      debug:
        msg: "File: {{ item.path }}, Size: {{ item.size }} bytes"
      loop: "{{ recent_files.files }}"
    
    - name: Archive files
      archive:
        path: /tmp/advanced-files/*.txt
        dest: /tmp/advanced-files/archive.tar.gz
        format: gz
    
    - name: Get archive information
      stat:
        path: /tmp/advanced-files/archive.tar.gz
      register: archive_stat
    
    - name: Display archive info
      debug:
        msg: "Archive size: {{ archive_stat.stat.size }} bytes"
    
    - name: Read first few lines of a file
      shell: head -n 3 /tmp/advanced-files/test1.txt
      register: file_head
      changed_when: false
    
    - name: Display file head
      debug:
        var: file_head.stdout_lines
EOF
```

### Step 10: Run Advanced Operations

```bash
ansible-playbook advanced-files.yml
```

## Expected Results

### File Creation

```
TASK [Create a directory] ****************************************************
changed: [localhost]

TASK [Create an empty file] **************************************************
changed: [localhost]
```

### File Stat Output

```json
{
    "stat": {
        "exists": true,
        "size": 0,
        "mode": "0644",
        "isdir": false,
        "isreg": true
    }
}
```

## File Module Reference

| Module | Purpose |
|--------|---------|
| `file` | Create/delete files, directories, symlinks |
| `copy` | Copy files with inline content or from source |
| `stat` | Get file information |
| `lineinfile` | Ensure line exists or modify line |
| `blockinfile` | Insert/update block of lines |
| `slurp` | Read file content (base64 encoded) |
| `find` | Search for files |
| `archive` | Create compressed archives |

## Troubleshooting

### Problem: Permission denied

**Solution:** Use `become: yes` or check file ownership
```yaml
- name: Create file with sudo
  file:
    path: /root/file.txt
    state: touch
  become: yes
```

### Problem: File already exists error

**Solution:** Use `force: yes` or check state
```yaml
- name: Force overwrite
  copy:
    src: file.txt
    dest: /tmp/file.txt
    force: yes
```

### Problem: Cannot modify file

**Solution:** Check if file is locked or has immutable attribute

## Additional Exercises

1. Create a directory structure with 10 nested levels
2. Copy multiple files using a loop
3. Modify configuration files using lineinfile
4. Set up a backup system using archive module
5. Create a playbook that manages file permissions for a web directory

## What You Learned

✅ Creating and managing files and directories  
✅ Using copy module for file content  
✅ Modifying file content with lineinfile and blockinfile  
✅ Managing file permissions (numeric and symbolic)  
✅ Using stat module to gather file information  
✅ Finding files with the find module  
✅ Creating archives  
✅ Reading file content with slurp  

## Next Steps

Move on to [Exercise 10: Package Management](../exercise-10/README.md) to learn how to install and manage software packages.
