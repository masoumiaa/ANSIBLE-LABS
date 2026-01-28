# Exercise 15: Vault Secrets

## Learning Objectives

- Encrypt sensitive data with Ansible Vault
- Create encrypted variable files
- Use vault passwords in playbooks
- Manage encrypted strings

## Prerequisites

- Completed Exercise 14
- Understanding of security basics

## Steps

### Step 1: Create Encrypted File

Create and encrypt a variable file:

```bash
# Create a vault password file
echo "my-vault-password-123" > /tmp/vault-password.txt
chmod 600 /tmp/vault-password.txt

# Create a file to encrypt
cat > /tmp/secrets.yml << 'EOF'
---
database_password: SuperSecretPassword123
api_key: abc123def456ghi789
jwt_secret: my-super-secret-jwt-key
aws_access_key: AKIAIOSFODNN7EXAMPLE
aws_secret_key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
EOF

# Encrypt the file
ansible-vault encrypt /tmp/secrets.yml --vault-password-file=/tmp/vault-password.txt

# View the encrypted file
cat /tmp/secrets.yml
```

### Step 2: View and Edit Encrypted Files

```bash
# View encrypted file
ansible-vault view /tmp/secrets.yml --vault-password-file=/tmp/vault-password.txt

# Edit encrypted file
# ansible-vault edit /tmp/secrets.yml --vault-password-file=/tmp/vault-password.txt

# Decrypt file (temporarily)
ansible-vault decrypt /tmp/secrets.yml --vault-password-file=/tmp/vault-password.txt

# Re-encrypt for next steps
ansible-vault encrypt /tmp/secrets.yml --vault-password-file=/tmp/vault-password.txt
```

### Step 3: Use Encrypted Variables in Playbook

Create a playbook using encrypted variables:

```bash
cat > use-vault.yml << 'EOF'
---
- name: Using Vault Encrypted Variables
  hosts: localhost
  connection: local
  vars_files:
    - /tmp/secrets.yml
  vars:
    app_name: SecureApp
  
  tasks:
    - name: Display non-sensitive information
      debug:
        msg: "Application: {{ app_name }}"
    
    - name: Use encrypted variables (don't display!)
      debug:
        msg: "Database password is configured (not shown for security)"
    
    - name: Create configuration with secrets
      copy:
        content: |
          # Application Configuration
          # SENSITIVE - DO NOT COMMIT
          
          [application]
          name={{ app_name }}
          
          [database]
          password={{ database_password }}
          
          [api]
          key={{ api_key }}
          jwt_secret={{ jwt_secret }}
          
          [aws]
          access_key={{ aws_access_key }}
          secret_key={{ aws_secret_key }}
        dest: /tmp/secure-config.ini
        mode: '0600'
    
    - name: Verify configuration file security
      stat:
        path: /tmp/secure-config.ini
      register: config_stat
    
    - name: Display file permissions
      debug:
        msg: "Config file permissions: {{ config_stat.stat.mode }}"
EOF
```

### Step 4: Run Playbook with Vault

```bash
# Run playbook with vault password file
ansible-playbook use-vault.yml --vault-password-file=/tmp/vault-password.txt

# Run with prompt for password
# ansible-playbook use-vault.yml --ask-vault-pass

# View the created config
cat /tmp/secure-config.ini
```

### Step 5: Encrypt Specific Strings

Create a playbook with encrypted strings:

```bash
# Encrypt a single string
encrypted_password=$(ansible-vault encrypt_string 'MySecretPassword' --name 'admin_password' --vault-password-file=/tmp/vault-password.txt)

# Create playbook with encrypted strings
cat > encrypted-strings.yml << 'EOF'
---
- name: Using Encrypted Strings
  hosts: localhost
  connection: local
  vars:
    # Regular variables
    app_name: MyApplication
    app_version: 1.0.0
    
    # Encrypted string (use ansible-vault encrypt_string)
    # admin_password: !vault |
    #     $ANSIBLE_VAULT;1.1;AES256
    #     ...encrypted content...
    admin_password: "PlainTextForDemo"  # In real use, encrypt this!
    
  tasks:
    - name: Display application info
      debug:
        msg: "{{ app_name }} v{{ app_version }}"
    
    - name: Create user configuration
      copy:
        content: |
          [user]
          name=admin
          password={{ admin_password }}
        dest: /tmp/user-config.ini
        mode: '0600'
    
    - name: Confirm configuration created
      debug:
        msg: "User configuration created securely"
EOF

# Run the playbook
ansible-playbook encrypted-strings.yml
```

### Step 6: Multiple Vault Passwords

Create files with different vault IDs:

```bash
# Create second vault password
echo "development-vault-pass" > /tmp/dev-vault-pass.txt
chmod 600 /tmp/dev-vault-pass.txt

echo "production-vault-pass" > /tmp/prod-vault-pass.txt
chmod 600 /tmp/prod-vault-pass.txt

# Create development secrets
cat > /tmp/dev-secrets.yml << 'EOF'
---
db_host: localhost
db_port: 5432
db_user: dev_user
db_password: dev_password_123
EOF

# Create production secrets
cat > /tmp/prod-secrets.yml << 'EOF'
---
db_host: prod-db.example.com
db_port: 5432
db_user: prod_user
db_password: super_secure_prod_pass_456
EOF

# Encrypt with vault IDs
ansible-vault encrypt /tmp/dev-secrets.yml --vault-id dev@/tmp/dev-vault-pass.txt
ansible-vault encrypt /tmp/prod-secrets.yml --vault-id prod@/tmp/prod-vault-pass.txt

# Create playbook using vault IDs
cat > multi-vault.yml << 'EOF'
---
- name: Multiple Vault Passwords
  hosts: localhost
  connection: local
  vars_files:
    - /tmp/dev-secrets.yml
  
  tasks:
    - name: Display database configuration
      debug:
        msg: |
          Host: {{ db_host }}
          Port: {{ db_port }}
          User: {{ db_user }}
          Password: [REDACTED]
    
    - name: Create database connection string
      set_fact:
        db_connection: "postgresql://{{ db_user }}:{{ db_password }}@{{ db_host }}:{{ db_port }}"
    
    - name: Confirm connection string created
      debug:
        msg: "Database connection configured"
EOF
```

### Step 7: Test Multiple Vaults

```bash
# Run with dev vault
ansible-playbook multi-vault.yml --vault-id dev@/tmp/dev-vault-pass.txt

# Switch to production secrets by editing vars_files in playbook, then:
# ansible-playbook multi-vault.yml --vault-id prod@/tmp/prod-vault-pass.txt
```

### Step 8: Vault Best Practices Demo

Create a secure deployment playbook:

```bash
# Create directory structure
mkdir -p /tmp/secure-deployment/{group_vars,host_vars}

# Create unencrypted common variables
cat > /tmp/secure-deployment/group_vars/all.yml << 'EOF'
---
# Common Variables (Unencrypted)
app_name: SecureApplication
app_port: 8080
log_level: INFO
backup_enabled: true
EOF

# Create encrypted secrets
cat > /tmp/secure-deployment/group_vars/vault.yml << 'EOF'
---
# Encrypted Secrets
vault_database_password: secret123
vault_api_key: apikey456
vault_ssl_key: sslkey789
EOF

# Encrypt the secrets file
ansible-vault encrypt /tmp/secure-deployment/group_vars/vault.yml --vault-password-file=/tmp/vault-password.txt

# Create deployment playbook
cat > secure-deployment.yml << 'EOF'
---
- name: Secure Deployment
  hosts: localhost
  connection: local
  vars_files:
    - /tmp/secure-deployment/group_vars/all.yml
    - /tmp/secure-deployment/group_vars/vault.yml
  
  tasks:
    - name: Display public configuration
      debug:
        msg: |
          Application: {{ app_name }}
          Port: {{ app_port }}
          Log Level: {{ log_level }}
    
    - name: Deploy application configuration
      copy:
        content: |
          # {{ app_name }} Configuration
          [app]
          name={{ app_name }}
          port={{ app_port }}
          log_level={{ log_level }}
          backup_enabled={{ backup_enabled }}
          
          [database]
          password={{ vault_database_password }}
          
          [api]
          key={{ vault_api_key }}
          
          [ssl]
          private_key={{ vault_ssl_key }}
        dest: /tmp/deployed-config.ini
        mode: '0600'
    
    - name: Create vault status report
      copy:
        content: |
          Secure Deployment Report
          ========================
          Date: {{ ansible_date_time.iso8601 }}
          
          Application: {{ app_name }}
          Port: {{ app_port }}
          
          Security:
          - Configuration file: /tmp/deployed-config.ini
          - File permissions: 0600 (owner read/write only)
          - Sensitive data: Encrypted with Ansible Vault
          - Vault-encrypted variables: 3
          
          Status: Deployed Successfully
        dest: /tmp/deployment-report.txt
    
    - name: Display deployment report
      command: cat /tmp/deployment-report.txt
      register: report
      changed_when: false
    
    - name: Show report
      debug:
        var: report.stdout_lines
EOF
```

### Step 9: Run Secure Deployment

```bash
ansible-playbook secure-deployment.yml --vault-password-file=/tmp/vault-password.txt
cat /tmp/deployment-report.txt
```

### Step 10: Vault Management Commands

Create a reference playbook for vault commands:

```bash
cat > vault-commands.yml << 'EOF'
---
- name: Vault Management Reference
  hosts: localhost
  connection: local
  gather_facts: no
  
  tasks:
    - name: Display vault management commands
      debug:
        msg: |
          Ansible Vault Commands:
          
          Create encrypted file:
            ansible-vault create secrets.yml
          
          Encrypt existing file:
            ansible-vault encrypt secrets.yml
          
          Decrypt file:
            ansible-vault decrypt secrets.yml
          
          View encrypted file:
            ansible-vault view secrets.yml
          
          Edit encrypted file:
            ansible-vault edit secrets.yml
          
          Rekey (change password):
            ansible-vault rekey secrets.yml
          
          Encrypt string:
            ansible-vault encrypt_string 'secret' --name 'var_name'
          
          Run playbook with vault:
            ansible-playbook playbook.yml --ask-vault-pass
            ansible-playbook playbook.yml --vault-password-file=pass.txt
            ansible-playbook playbook.yml --vault-id @pass.txt
    
    - name: Display vault best practices
      debug:
        msg: |
          Best Practices:
          
          1. Never commit unencrypted secrets to version control
          2. Use separate files for encrypted data (vault.yml)
          3. Keep vault password files secure and out of repos
          4. Use vault IDs for multiple environments
          5. Set proper file permissions (0600) for sensitive files
          6. Use ansible-vault edit instead of decrypt/encrypt
          7. Consider using external secret management (HashiCorp Vault, AWS Secrets Manager)
          8. Regularly rotate vault passwords
          9. Use different vault passwords for different environments
          10. Document which variables are encrypted
EOF

ansible-playbook vault-commands.yml
```

## Expected Results

### Encrypted File Content

```
$ANSIBLE_VAULT;1.1;AES256
66386439653361353038653534343265383431396330346334663161656138373464346537303436
3961633264326365623566323235656539346232663733310a323961333635313630316138623034
...
```

### Viewing Decrypted Content

```yaml
---
database_password: SuperSecretPassword123
api_key: abc123def456ghi789
```

## Ansible Vault Commands

| Command | Description |
|---------|-------------|
| `ansible-vault create` | Create new encrypted file |
| `ansible-vault encrypt` | Encrypt existing file |
| `ansible-vault decrypt` | Decrypt file |
| `ansible-vault view` | View encrypted file |
| `ansible-vault edit` | Edit encrypted file |
| `ansible-vault rekey` | Change vault password |
| `ansible-vault encrypt_string` | Encrypt single string |

## Vault Password Options

```bash
# Prompt for password
--ask-vault-pass

# Use password file
--vault-password-file=file.txt

# Use vault ID
--vault-id dev@pass.txt

# Multiple vault IDs
--vault-id dev@pass1.txt --vault-id prod@pass2.txt
```

## Troubleshooting

### Problem: "Decryption failed"

**Solution:** Check vault password is correct
```bash
ansible-vault view secrets.yml --ask-vault-pass
```

### Problem: "Vault password file not found"

**Solution:** Use absolute path
```bash
--vault-password-file=/full/path/to/password.txt
```

### Problem: Cannot edit encrypted file

**Solution:** Use ansible-vault edit
```bash
ansible-vault edit secrets.yml --vault-password-file=pass.txt
```

## Additional Exercises

1. Create encrypted files for dev, staging, and production
2. Use vault IDs for multi-environment deployment
3. Encrypt individual variables in group_vars
4. Create a secure password rotation playbook
5. Integrate vault with a secrets management service

## What You Learned

✅ Encrypting files with Ansible Vault  
✅ Viewing and editing encrypted files  
✅ Using encrypted variables in playbooks  
✅ Encrypting individual strings  
✅ Managing multiple vault passwords with vault IDs  
✅ Vault best practices and security patterns  
✅ Secure file permissions for sensitive data  
✅ Vault command reference  

## Next Steps

Move on to [Exercise 16: Roles Basics](../exercise-16/README.md) to learn about organizing playbooks with roles.
