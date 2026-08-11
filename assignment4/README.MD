# Install Nginx, Change Port and Deploy Application Using Ansible

## Objective

The objective of this assignment is to:

1. Install Nginx using Ansible.
2. Change the default Nginx port from `80` to `8081`.
3. Deploy a simple web application.
4. Start and enable the Nginx service.

## Project Structure

```text
assignment4/
├── ansible.cfg
├── host
├── nginx.yml
├── README.md
└── .gitignore
```

## Step 1: Test Ansible Connectivity

Run:

```bash
ansible all -m ping
```

Expected output:

```text
server1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## Step 2: Create the Nginx Playbook

Create the playbook:

```bash
nano nginx.yml
```

The playbook performs the following tasks:

* Installs Nginx.
* Changes the Nginx port from 80 to 8081.
* Deploys an HTML application.
* Restarts Nginx.
* Enables Nginx at system startup.

## Step 3: Check Playbook Syntax

Run:

```bash
ansible-playbook --syntax-check nginx.yml
```

Expected output:

```text
playbook: nginx.yml
```

## Step 4: Run the Playbook

Run:

```bash
ansible-playbook nginx.yml
```

## Step 5: Verify Nginx Service

Connect to the target instance:

```bash
ssh -i recovery-key.pem ubuntu@<EC2-PUBLIC-IP>
```

Check the service:

```bash
sudo systemctl status nginx
```

## Step 6: Verify Port 8081

Run:

```bash
sudo ss -tlnp | grep 8081
```

Expected:

```text
LISTEN 0 511 0.0.0.0:8081
```

## Step 7: Test the Application

Run:

```bash
curl http://localhost:8081
```

Expected:

```text
Welcome to My Application
This application was deployed using Ansible.
Nginx is running on port 8081.
```

## Step 8: Configure AWS Security Group

Allow inbound TCP traffic on port `8081` in the EC2 Security Group.

Then access the application using:

```text
http://<EC2-PUBLIC-IP>:8081
```

## Result

Nginx was successfully installed using Ansible.

The default port was changed:

```text
80 → 8081
```

A sample web application was deployed to:

```text
/var/www/html/index.html
```

The application can be accessed using:

```text
http://<EC2-PUBLIC-IP>:8081
```

## Conclusion

The Ansible playbook successfully automated Nginx installation, port configuration, application deployment, and service management.
