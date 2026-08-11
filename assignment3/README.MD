# Ansible Dynamic Inventory with AWS EC2

## Objective

The objective of this assignment is to configure **Ansible Dynamic Inventory** with AWS EC2 instances.

Dynamic inventory allows Ansible to automatically discover EC2 instances from AWS instead of manually maintaining IP addresses in a static inventory file.

---

## Architecture

```text
AWS EC2 Instances
       |
       ↓
AWS EC2 Dynamic Inventory Plugin
       |
       ↓
Ansible Controller
       |
       ↓
Ansible Playbook
```

---

## Prerequisites

* AWS account
* AWS EC2 instance
* Ubuntu Ansible Controller
* Ansible installed
* AWS CLI installed
* IAM permissions to describe EC2 instances
* SSH private key

---

## Step 1: Install AWS Collection

Install the Amazon AWS Ansible collection:

```bash
ansible-galaxy collection install amazon.aws
```

Install the required Python packages:

```bash
sudo apt update
sudo apt install python3-boto3 python3-botocore -y
```

Verify:

```bash
ansible-galaxy collection list | grep amazon.aws
```

---

## Step 2: Configure AWS CLI

Configure AWS credentials:

```bash
aws configure
```

Enter:

```text
AWS Access Key ID:
AWS Secret Access Key:
Default region name: us-east-1
Default output format: json
```

Verify AWS access:

```bash
aws sts get-caller-identity
```

If the command returns the AWS account and IAM identity information, AWS authentication is working.

> For production environments, an IAM role attached to the Ansible controller is preferred over storing long-lived AWS access keys.

---

## Step 3: Create Dynamic Inventory File

Go to the assignment directory:

```bash
cd /home/ubuntu/Ansible-assignment/assignment4
```

Create the dynamic inventory file:

```bash
nano aws_ec2.yml
```

Add:

```yaml
---
plugin: amazon.aws.aws_ec2

regions:
  - us-east-1

filters:
  instance-state-name: running

hostnames:
  - public-ip-address
```

### Explanation

| Configuration | Purpose                                          |
| ------------- | ------------------------------------------------ |
| `plugin`      | Uses the AWS EC2 inventory plugin                |
| `regions`     | Specifies the AWS region                         |
| `filters`     | Selects only running instances                   |
| `hostnames`   | Uses the EC2 public IP as the inventory hostname |

---

## Step 4: Verify Dynamic Inventory

Display the complete inventory:

```bash
ansible-inventory -i aws_ec2.yml --list
```

Display the inventory as a tree:

```bash
ansible-inventory -i aws_ec2.yml --graph
```

Example:

```text
@all:
  |--@ungrouped:
  |  |--54.123.45.67
```

The EC2 instance is discovered automatically from AWS.

---

## Step 5: Test Connectivity

Run:

```bash
ansible -i aws_ec2.yml all -m ping
```

Expected:

```text
54.123.45.67 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

This confirms that Ansible can discover and connect to the EC2 instance.

---

## Step 6: Create a Playbook

Create an Nginx playbook:

```bash
nano nginx.yml
```

Add:

```yaml
---
- name: Install Nginx on AWS EC2 instances
  hosts: all
  become: true

  tasks:

    - name: Install Nginx
      apt:
        name: nginx
        state: present
        update_cache: true

    - name: Start and enable Nginx
      service:
        name: nginx
        state: started
        enabled: true
```

---

## Step 7: Check Playbook Syntax

Run:

```bash
ansible-playbook -i aws_ec2.yml --syntax-check nginx.yml
```

Expected:

```text
playbook: nginx.yml
```

---

## Step 8: Run the Playbook

Run:

```bash
ansible-playbook -i aws_ec2.yml nginx.yml
```

Ansible will automatically discover the running EC2 instances and install Nginx on them.

---

## Step 9: Dynamic Inventory Using EC2 Tags

Dynamic inventory can also create Ansible groups from EC2 tags.

For example, if an EC2 instance has:

```text
Environment = production
```

Modify `aws_ec2.yml`:

```yaml
---
plugin: amazon.aws.aws_ec2

regions:
  - us-east-1

filters:
  instance-state-name: running

hostnames:
  - public-ip-address

keyed_groups:
  - key: tags.Environment
    prefix: environment
    separator: "_"
```

Check the inventory:

```bash
ansible-inventory -i aws_ec2.yml --graph
```

Example:

```text
@all:
  |--@aws_ec2:
  |  |--54.123.45.67
  |
  |--@environment_production:
  |  |--54.123.45.67
```

Now you can target only production instances:

```yaml
---
- name: Configure Production Servers
  hosts: environment_production
  become: true

  tasks:

    - name: Install Nginx
      apt:
        name: nginx
        state: present
```

Run:

```bash
ansible-playbook -i aws_ec2.yml nginx.yml
```

---

## Important Commands

### Install AWS collection

```bash
ansible-galaxy collection install amazon.aws
```

### Check AWS authentication

```bash
aws sts get-caller-identity
```

### Display dynamic inventory

```bash
ansible-inventory -i aws_ec2.yml --list
```

### Display inventory graph

```bash
ansible-inventory -i aws_ec2.yml --graph
```

### Test EC2 connectivity

```bash
ansible -i aws_ec2.yml all -m ping
```

### Run a playbook

```bash
ansible-playbook -i aws_ec2.yml nginx.yml
```

---

## Project Structure

```text
assignment4/
├── ansible.cfg
├── aws_ec2.yml
├── nginx.yml
├── README.md
└── .gitignore
```

---

## GitHub Upload

Before uploading, make sure the private key is excluded.

Create `.gitignore`:

```bash
nano .gitignore
```

Add:

```text
*.pem
```

Check:

```bash
git status
```

Add the project files:

```bash
git add aws_ec2.yml nginx.yml README.md .gitignore
```

Commit:

```bash
git commit -m "Add AWS dynamic inventory"
```

Push:

```bash
git push origin main
```

---

## Result

The AWS EC2 instances are automatically discovered by Ansible using the dynamic inventory plugin.

```text
AWS EC2
   ↓
Dynamic Inventory
   ↓
EC2 Instances Automatically Discovered
   ↓
Ansible Ping
   ↓
Ansible Playbook
   ↓
Nginx Installation
```

## Conclusion

Ansible Dynamic Inventory eliminates the need to manually maintain EC2 IP addresses in a static inventory file. It is especially useful in cloud environments where instances are frequently created, terminated, or changed.
