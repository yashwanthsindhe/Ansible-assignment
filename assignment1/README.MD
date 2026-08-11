# Tomcat Application Deployment Using Ansible Roles

## Objective

The objective of this assignment is to deploy a new version of a Tomcat WAR application using **Ansible Roles**.

The deployment process performs the following steps:

```text
Stop Tomcat
     ↓
Backup Existing WAR
     ↓
Deploy New WAR
     ↓
Start Tomcat
     ↓
Check Application Health
```

---

## Project Structure

```text
assignment4/
├── ansible.cfg
├── host
├── deploy.yml
├── myapp.war
├── README.md
└── roles/
    └── deployment/
        ├── tasks/
        │   └── main.yml
        ├── handlers/
        │   └── main.yml
        ├── files/
        │   └── myapp.war
        ├── templates/
        ├── vars/
        │   └── main.yml
        └── defaults/
            └── main.yml
```

---

# Step 1: Create the Role

Create the deployment role:

```bash
ansible-galaxy init roles/deployment
```

Verify:

```bash
ls roles/deployment
```

You should see:

```text
defaults
files
handlers
meta
README.md
tasks
templates
tests
vars
```

---

# Step 2: Add the WAR File

Copy the new WAR file into the role's `files` directory:

```bash
cp myapp.war roles/deployment/files/
```

Verify:

```bash
ls -lh roles/deployment/files/
```

Expected:

```text
myapp.war
```

---

# Step 3: Create Role Tasks

Edit:

```bash
nano roles/deployment/tasks/main.yml
```

Add:

```yaml
---
- name: Stop Tomcat
  service:
    name: tomcat10
    state: stopped

- name: Create backup directory
  file:
    path: /opt/tomcat/backup
    state: directory
    mode: '0755'

- name: Check if existing WAR file exists
  stat:
    path: /opt/tomcat/webapps/myapp.war
  register: existing_war

- name: Backup existing WAR file
  copy:
    src: /opt/tomcat/webapps/myapp.war
    dest: /opt/tomcat/backup/myapp.war.backup
    remote_src: true
  when: existing_war.stat.exists

- name: Remove old WAR file
  file:
    path: /opt/tomcat/webapps/myapp.war
    state: absent

- name: Deploy new WAR file
  copy:
    src: myapp.war
    dest: /opt/tomcat/webapps/myapp.war
    owner: tomcat
    group: tomcat
    mode: '0644'

- name: Start Tomcat
  service:
    name: tomcat10
    state: started
    enabled: true

- name: Wait for Tomcat port
  wait_for:
    host: 127.0.0.1
    port: 9090
    delay: 5
    timeout: 60

- name: Check application health
  uri:
    url: http://127.0.0.1:9090/myapp/
    status_code: 200
  register: application_health

- name: Display application health
  debug:
    msg: "Application is healthy. HTTP status: {{ application_health.status }}"
```

---

# Step 4: Create the Main Playbook

Create:

```bash
nano deploy.yml
```

Add:

```yaml
---
- name: Deploy Tomcat Application
  hosts: all
  become: true

  roles:
    - deployment
```

The playbook calls the `deployment` role.

---

# Step 5: Check Role Syntax

Run:

```bash
ansible-playbook --syntax-check deploy.yml
```

Expected:

```text
playbook: deploy.yml
```

---

# Step 6: Test Ansible Connectivity

```bash
ansible all -m ping
```

Expected:

```text
server1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

---

# Step 7: Run the Deployment

Run:

```bash
ansible-playbook deploy.yml
```

The role performs:

```text
Stop Tomcat
     ↓
Create Backup Directory
     ↓
Backup Existing WAR
     ↓
Remove Old WAR
     ↓
Deploy New WAR
     ↓
Start Tomcat
     ↓
Wait for Port 9090
     ↓
Health Check
```

---

# Step 8: Verify Tomcat

On the target server:

```bash
sudo systemctl status tomcat10
```

Expected:

```text
Active: active (running)
```

---

# Step 9: Verify Tomcat Port

```bash
sudo ss -tlnp | grep 9090
```

Expected:

```text
LISTEN 0 100 0.0.0.0:9090
```

---

# Step 10: Verify Backup

Check the backup directory:

```bash
ls -lh /opt/tomcat/backup/
```

Expected:

```text
myapp.war.backup
```

---

# Step 11: Verify New WAR

```bash
ls -lh /opt/tomcat/webapps/
```

Expected:

```text
myapp.war
```

---

# Step 12: Check Application Health

Run:

```bash
curl -I http://localhost:9090/myapp/
```

Expected:

```text
HTTP/1.1 200
```

The Ansible `uri` task also performs this health check automatically.

Example:

```text
Application is healthy. HTTP status: 200
```

---

# Deployment Role Advantages

Using an Ansible Role provides:

* Reusable deployment logic
* Organized project structure
* Easy maintenance
* Separation of tasks
* Easy rollback
* Suitable for organization-level automation

---

# Deployment Flow

```text
                 Ansible Controller
                        |
                        ↓
                  deploy.yml
                        |
                        ↓
                deployment role
                        |
          ┌─────────────┴─────────────┐
          ↓                           ↓
    Stop Tomcat                  Backup WAR
          |                           |
          └─────────────┬─────────────┘
                        ↓
                  Deploy New WAR
                        |
                        ↓
                   Start Tomcat
                        |
                        ↓
                Health Check :9090
                        |
                        ↓
                     HTTP 200
```

---

# Result

The Tomcat application was successfully deployed using an Ansible Role.

The deployment process:

* Stopped the Tomcat service.
* Backed up the existing WAR file.
* Deployed the new WAR file.
* Started Tomcat.
* Verified the Tomcat port.
* Checked the health of the deployed application.

## Conclusion

Ansible Roles provide a clean and reusable way to automate application deployments. The role can be reused for future WAR deployments by changing the application name or WAR file.
