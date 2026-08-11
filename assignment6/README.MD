# Ansible Target Instances Report

## Objective

The objective of this assignment is to use Ansible to collect system information from target instances.

The report contains:

* CPU
* RAM
* Storage
* Hostname

## Project Structure

```text
assignment4/
├── ansible.cfg
├── host
├── report.yml
├── README.md
└── recovery-key.pem
```

> **Important:** Do not upload `recovery-key.pem` to GitHub.

## Ansible Configuration

The `ansible.cfg` file contains the inventory location, remote user, SSH host key configuration, and private key location.

Example:

```ini
[defaults]
inventory = /home/ubuntu/Ansible-assignment/assignment4/host
remote_user = ubuntu
host_key_checking = False
private_key_file = /home/ubuntu/Ansible-assignment/assignment4/recovery-key.pem
```

## Inventory

The `host` file contains the target instances.

Example:

```ini
[targets]
server1 ansible_host=YOUR_EC2_PUBLIC_IP
server2 ansible_host=YOUR_EC2_PUBLIC_IP
```

## Playbook

The `report.yml` playbook uses Ansible gathered facts to collect:

| Information | Ansible Variable               |
| ----------- | ------------------------------ |
| Hostname    | `ansible_hostname`             |
| CPU         | `ansible_processor_vcpus`      |
| RAM         | `ansible_memtotal_mb`          |
| Storage     | `ansible_mounts[0].size_total` |

## Step 1: Test Ansible Connectivity

Run:

```bash
ansible all -m ping
```

Expected result:

```text
SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## Step 2: Check Playbook Syntax

Run:

```bash
ansible-playbook --syntax-check report.yml
```

Expected:

```text
playbook: report.yml
```

## Step 3: Run the Playbook

Run:

```bash
ansible-playbook report.yml
```

## Sample Output

```text
======================================
       TARGET INSTANCE REPORT
======================================
Hostname : ip-172-31-20-15
CPU      : 2 cores
RAM      : 1950 MB
Storage  : 20.00 GB
======================================
```

## Technologies Used

* Ansible
* Ubuntu
* AWS EC2
* YAML
* Linux

## Conclusion

The Ansible playbook successfully collects CPU, RAM, Storage, and Hostname information from the target instances and displays it as a system report.
