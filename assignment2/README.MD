# Ansible Vault

## Objective

The objective of this assignment is to use **Ansible Vault** to securely store sensitive information such as passwords, API keys, database credentials, and other secrets.

Ansible Vault encrypts files so that sensitive information is not stored as plain text.

---

## What is Ansible Vault?

Ansible Vault is a feature of Ansible used to encrypt sensitive data.

Examples of information that can be protected:

* Database passwords
* Application passwords
* API keys
* AWS credentials
* SSH-related secrets
* User passwords
* Environment variables

Instead of storing secrets like this:

```yaml
db_password: MySecretPassword123
```

we can encrypt the file using Ansible Vault.

---

## Project Structure

```text
assignment4/
├── ansible.cfg
├── host
├── secrets.yml
├── vault-example.yml
├── README.md
└── .gitignore
```

---

# Step 1: Create the Ansible Vault File

Create an encrypted file:

```bash
ansible-vault create secrets.yml
```

Ansible will ask for a Vault password.

Enter the password and then add:

```yaml
db_username: admin
db_password: MySecretPassword123
```

Save and exit.

The file `secrets.yml` is now encrypted.

---

# Step 2: View the Encrypted File

Run:

```bash
cat secrets.yml
```

You will see something similar to:

```text
$ANSIBLE_VAULT;1.1;AES256
613532633231343563...
...
```

The actual secret is no longer visible as plain text.

---

# Step 3: View the Decrypted Content

Run:

```bash
ansible-vault view secrets.yml
```

Enter the Vault password.

You will see:

```yaml
db_username: admin
db_password: MySecretPassword123
```

---

# Step 4: Create a Playbook

Create:

```bash
nano vault-example.yml
```

Add:

```yaml
---
- name: Ansible Vault Example
  hosts: all
  become: true

  vars_files:
    - secrets.yml

  tasks:

    - name: Display database username
      debug:
        msg: "Database username is {{ db_username }}"

    - name: Display database password
      debug:
        msg: "Database password is {{ db_password }}"
```

---

# Step 5: Run the Playbook

Run:

```bash
ansible-playbook vault-example.yml --ask-vault-pass
```

Ansible will ask:

```text
Vault password:
```

Enter the password used when creating `secrets.yml`.

Example output:

```text
TASK [Display database username]
ok: [server1] => {
    "msg": "Database username is admin"
}

TASK [Display database password]
ok: [server1] => {
    "msg": "Database password is MySecretPassword123"
}
```

---

# Step 6: Hide Sensitive Output

It is not recommended to display passwords in the Ansible output.

Use `no_log: true` when handling sensitive values.

Example:

```yaml
---
- name: Ansible Vault Secure Example
  hosts: all
  become: true

  vars_files:
    - secrets.yml

  tasks:

    - name: Use database password securely
      debug:
        msg: "Database credentials loaded securely"
      no_log: true
```

---

# Step 7: Edit an Existing Vault File

To modify the encrypted file:

```bash
ansible-vault edit secrets.yml
```

Enter the Vault password.

Make your changes and save the file.

---

# Step 8: Change the Vault Password

To change the Vault password:

```bash
ansible-vault rekey secrets.yml
```

Enter:

```text
Vault password:
New Vault password:
Confirm New Vault password:
```

---

# Step 9: Encrypt an Existing File

If you already have a normal YAML file:

```bash
nano secrets.yml
```

For example:

```yaml
db_username: admin
db_password: MySecretPassword123
```

Encrypt it:

```bash
ansible-vault encrypt secrets.yml
```

---

# Step 10: Decrypt the Vault File

To permanently decrypt the file:

```bash
ansible-vault decrypt secrets.yml
```

The file will become readable YAML again.

---

# Step 11: Check Vault File

You can check whether a file is encrypted using:

```bash
head -n 1 secrets.yml
```

Expected:

```text
$ANSIBLE_VAULT;1.1;AES256
```

---

# Important Ansible Vault Commands

| Command                                          | Purpose                 |
| ------------------------------------------------ | ----------------------- |
| `ansible-vault create secrets.yml`               | Create encrypted file   |
| `ansible-vault view secrets.yml`                 | View encrypted file     |
| `ansible-vault edit secrets.yml`                 | Edit encrypted file     |
| `ansible-vault encrypt secrets.yml`              | Encrypt existing file   |
| `ansible-vault decrypt secrets.yml`              | Decrypt file            |
| `ansible-vault rekey secrets.yml`                | Change Vault password   |
| `ansible-playbook playbook.yml --ask-vault-pass` | Run playbook with Vault |

---

# Step 12: Git Configuration

Create a `.gitignore` file:

```bash
nano .gitignore
```

For this assignment, you can keep the encrypted Vault file in Git because the contents are encrypted.

However, never commit:

* Vault passwords
* Unencrypted secret files
* Private SSH keys
* AWS access keys

For example:

```text
*.pem
vault-password.txt
```

---

# Step 13: GitHub Upload

Check the files:

```bash
git status
```

Add the assignment:

```bash
git add secrets.yml vault-example.yml README.md .gitignore
```

Commit:

```bash
git commit -m "Add Ansible Vault example"
```

Push:

```bash
git push origin main
```

---

# Example Workflow

```text
Create secrets
      ↓
ansible-vault create secrets.yml
      ↓
Store passwords/API keys
      ↓
Vault encrypts the file
      ↓
Reference secrets.yml in playbook
      ↓
Run playbook
      ↓
ansible-playbook vault-example.yml --ask-vault-pass
```

# Result

Ansible Vault successfully encrypts sensitive information and allows the encrypted variables to be securely used inside Ansible playbooks.

## Conclusion

Ansible Vault is useful in organization-level automation because passwords, database credentials, API keys, and other sensitive information should not be stored as plain text in Ansible playbooks or repositories.
