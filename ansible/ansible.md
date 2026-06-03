# Ansible Lab

## Objective

* Create an Ansible Controller and Managed Nodes
* Configure one AWS Linux Controller and Three Managed Nodes
* Test Ansible connectivity using SSH
* Execute ad-hoc commands from the controller

## Lab Architecture

| Server     | OS                       | Role                 |
| ---------- | ------------------------ | -------------------- |
| Controller | Amazon Linux 2023        | Ansible Control Node |
| Node-1     | SUSE Linux               | Managed Node         |
| Node-1     | Ubuntu                   | Managed Node         |
| Node-2     | Dabian                   | Managed Node         |

---

## Install Ansible

### Controller (Ubuntu)

```bash
sudo apt update
sudo apt install ansible -y
```
! different OS machine run on different install command. so, best checking on net first before running it.


### Verify Installation

```bash
ansible --version
```

---

## Create Ansible User

### Node-1 (RHEL)
* creating a user
* adding password to that user
* giving nopassword permission 
```
sudo useradd ansible
sudo passwd ansible
sudo visudo
<ansible ALL=(ALL) NOPASSWD: ALL>
```

### Node-2 (Amazon Linux 2023)
* creating a user
* adding password to that user
* giving nopassword permission 
```
sudo useradd ansible
sudo passwd ansible
sudo visudo
<ansible ALL=(ALL) NOPASSWD: ALL>
```

* now, need to find sshd.config file. (usually located in /ect/ssh/sshd_config/)
* to change password authentication from no to yes
* one of the challenge different OS machine has different location for this line
* # it is important to remember that only find and change password authentication only. anything other changed might trigger error
```
Password Authentication: No
         ||
         \/
Password Authentication: Yes
 ```
* then, restart the sshd
```
sudo systemctl restart sshd
```
---

## Configure SSH Key Authentication

### Generate SSH Key on Controller

```bash
ssh-keygen
```

### Copy Public Key to Nodes

```bash
ssh-copy-id ansible@<node-1-private-ip>

ssh-copy-id ansible@<node-2-private-ip>
```

### Verify SSH Access

```bash
ssh ansible@<node-1-private-ip>

ssh ansible@<node-2-private-ip>
```

---

## Create ansible inventory
* make ansible directory
* inside that, make ansible.config and hosts file
  
in hosts:

```
[node]
172.31.xx.xx
172.31.xx.xx

[node:vars]
ansible_user=ansible
```
in ansible.cfg
```
[defaults]
inventory = /home/ansible/ansible/hosts
```
---

## Test Connectivity

```bash
ansible all -m ping
```

Expected Result:

```text
SUCCESS
```

---

## Execute Ad-Hoc Commands

Check Hostname:

```bash
ansible all -a "hostname"
```

Check Disk Usage:

```bash
ansible all -a "df -h"
```

Check Uptime:

```bash
ansible all -a "uptime"
```

---

## Issue Encountered

Ansible ping initially failed with:

```text
Permission denied
```

### Cause

Ansible attempted to connect using the controller username (`ubuntu`) instead of the managed node user (`ansible`).

### Fix

Added:

```ini
[node:vars]
ansible_user=ansible
```

to the inventory file.

---

## Outcome

Successfully managed Red Hat and Amazon Linux servers from a single Ubuntu controller using Ansible.

