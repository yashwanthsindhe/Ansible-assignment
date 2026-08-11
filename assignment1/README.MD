# Tomcat Application Deployment Using Ansible

## Objective

The objective of this deployment is to safely deploy a new version of a WAR application on a Tomcat server.

The deployment process follows these steps:

```text
Stop Tomcat
     ↓
Backup Existing WAR File
     ↓
Deploy New WAR File
     ↓
Start Tomcat
     ↓
Check Application Health
```

---

## Deployment Steps

### Step 1: Stop Tomcat Service

Before deploying the new application, stop the Tomcat service.

```bash
sudo systemctl stop tomcat10
```

Verify that Tomcat has stopped:

```bash
sudo systemctl status tomcat10
```

Expected:

```text
Active: inactive (dead)
```

---

### Step 2: Backup Existing Application

Create a backup directory:

```bash
sudo mkdir -p /opt/tomcat/backup
```

Copy the existing WAR file to the backup directory:

```bash
sudo cp /opt/tomcat/webapps/myapp.war /opt/tomcat/backup/myapp.war
```

Verify the backup:

```bash
ls -lh /opt/tomcat/backup/
```

Example:

```text
-rw-r--r-- 1 root root 25M Aug 11 15:30 myapp.war
```

The existing application is now backed up before deploying the new version.

---

### Step 3: Deploy New WAR File

Copy the new WAR file to the Tomcat `webapps` directory:

```bash
sudo cp myapp.war /opt/tomcat/webapps/myapp.war
```

Verify:

```bash
ls -lh /opt/tomcat/webapps/myapp.war
```

---

### Step 4: Start Tomcat

Start the Tomcat service:

```bash
sudo systemctl start tomcat10
```

Enable Tomcat at system startup:

```bash
sudo systemctl enable tomcat10
```

Check the service:

```bash
sudo systemctl status tomcat10
```

Expected:

```text
Active: active (running)
```

---

### Step 5: Check Application Health

Check whether Tomcat is listening on its configured port:

```bash
sudo ss -tlnp | grep 9090
```

Test the application locally:

```bash
curl -I http://localhost:9090/myapp/
```

Expected:

```text
HTTP/1.1 200
```

A `200 OK` response indicates that the application is responding successfully.

You can also access the application from a browser:

```text
http://<EC2-PUBLIC-IP>:9090/myapp/
```

---

# Ansible Deployment Playbook

Create the deployment playbook:

```bash
nano deploy.yml
```

Use:

```yaml
---
- name: Deploy new Tomcat application
  hosts: all
  become: true

  vars:
    tomcat_service: tomcat10
    tomcat_webapps: /opt/tomcat/webapps
    backup_dir: /opt/tomcat/backup
    app_name: myapp
    war_file: myapp.war
    tomcat_port: 9090

  tasks:

    - name: Stop Tomcat
      service:
        name: "{{ tomcat_service }}"
        state: stopped

    - name: Create backup directory
      file:
        path: "{{ backup_dir }}"
        state: directory
        mode: '0755'

    - name: Backup existing WAR file
      copy:
        src: "{{ tomcat_webapps }}/{{ war_file }}"
        dest: "{{ backup_dir }}/{{ war_file }}.backup"
        remote_src: true
      when: >
        (tomcat_webapps + '/' + war_file) is file

    - name: Remove old WAR file
      file:
        path: "{{ tomcat_webapps }}/{{ war_file }}"
        state: absent

    - name: Deploy new WAR file
      copy:
        src: "{{ war_file }}"
        dest: "{{ tomcat_webapps }}/{{ war_file }}"
        owner: tomcat
        group: tomcat
        mode: '0644'

    - name: Start Tomcat
      service:
        name: "{{ tomcat_service }}"
        state: started
        enabled: true

    - name: Wait for Tomcat port
      wait_for:
        host: 127.0.0.1
        port: "{{ tomcat_port }}"
        delay: 5
        timeout: 60

    - name: Check application health
      uri:
        url: "http://127.0.0.1:{{ tomcat_port }}/{{ app_name }}/"
        status_code: 200
      register: health_check

    - name: Display health status
      debug:
        msg: "Application {{ app_name }} is healthy and responding with HTTP {{ health_check.status }}"
```

---

# Playbook Execution

Before running the deployment, make sure the new WAR file is available on the Ansible controller:

```bash
ls -lh myapp.war
```

Check the playbook syntax:

```bash
ansible-playbook --syntax-check deploy.yml
```

Run the deployment:

```bash
ansible-playbook deploy.yml
```

---

# Deployment Flow

```text
              New WAR File
                   |
                   ↓
            Stop Tomcat
                   |
                   ↓
       Backup Existing WAR
                   |
                   ↓
          Remove Old WAR
                   |
                   ↓
          Deploy New WAR
                   |
                   ↓
            Start Tomcat
                   |
                   ↓
          Wait for Port 9090
                   |
                   ↓
       Application Health Check
                   |
                   ↓
              HTTP 200 OK
```

---

# Verification Commands

### Check Tomcat

```bash
sudo systemctl status tomcat10
```

### Check Tomcat port

```bash
sudo ss -tlnp | grep 9090
```

### Check deployed WAR

```bash
ls -lh /opt/tomcat/webapps/
```

### Check backup

```bash
ls -lh /opt/tomcat/backup/
```

### Check application

```bash
curl -I http://localhost:9090/myapp/
```

---

# Rollback

If the new application fails, stop Tomcat:

```bash
sudo systemctl stop tomcat10
```

Remove the new WAR:

```bash
sudo rm -f /opt/tomcat/webapps/myapp.war
```

Restore the backup:

```bash
sudo cp /opt/tomcat/backup/myapp.war.backup /opt/tomcat/webapps/myapp.war
```

Start Tomcat:

```bash
sudo systemctl start tomcat10
```

Check:

```bash
sudo systemctl status tomcat10
```

Test:

```bash
curl -I http://localhost:9090/myapp/
```

---

# Result

The deployment successfully performs the following operations:

1. Stops Tomcat.
2. Creates a backup of the existing WAR file.
3. Removes the old application.
4. Deploys the new WAR file.
5. Starts Tomcat.
6. Waits for the Tomcat port.
7. Performs an application health check.
8. Confirms the application with an HTTP `200 OK` response.

## Conclusion

This deployment process provides a simple and safe way to deploy a new version of a Tomcat application using Ansible while maintaining a backup for rollback.
