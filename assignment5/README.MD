# Install Tomcat and Change Port 8080 to 9090 Using Ansible

## Objective

The objective of this assignment is to install Apache Tomcat on the target EC2 instance using Ansible and change the default Tomcat port from `8080` to `9090`.

## Prerequisites

* Ubuntu Ansible Controller
* Target Ubuntu EC2 instance
* Ansible installed
* SSH private key
* Java
* AWS Security Group access

## Project Structure

```text
assignment4/
├── ansible.cfg
├── host
├── tomcat.yml
├── README.md
└── .gitignore
```

## Step 1: Test Connectivity

Run:

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

## Step 2: Create Tomcat Playbook

Create the playbook:

```bash
nano tomcat.yml
```

The playbook performs the following tasks:

1. Installs Java.
2. Installs Tomcat 10.
3. Changes the Tomcat port from 8080 to 9090.
4. Restarts Tomcat.
5. Enables Tomcat to start automatically after reboot.

## Step 3: Validate the Playbook

```bash
ansible-playbook --syntax-check tomcat.yml
```

Expected:

```text
playbook: tomcat.yml
```

## Step 4: Run the Playbook

```bash
ansible-playbook tomcat.yml
```

## Step 5: Verify Tomcat Service

On the target instance:

```bash
sudo systemctl status tomcat10
```

## Step 6: Verify Port 9090

Run:

```bash
sudo ss -tlnp | grep 9090
```

Expected:

```text
LISTEN 0 100 0.0.0.0:9090
```

## Step 7: Test Tomcat

Run:

```bash
curl http://localhost:9090
```

## Step 8: Access Tomcat from Browser

Open:

```text
http://<EC2-PUBLIC-IP>:9090
```

Make sure TCP port `9090` is allowed in the EC2 Security Group.

## Result

Tomcat was successfully installed using Ansible, and the default port was changed from:

```text
8080 → 9090
```

Tomcat can now be accessed using:

```text
http://<EC2-PUBLIC-IP>:9090
```

## Conclusion

The Ansible playbook automates Tomcat installation, configuration, service management, and port modification on the target EC2 instances.
