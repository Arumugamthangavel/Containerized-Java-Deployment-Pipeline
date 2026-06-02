# Ansible Lab

## Objective

* Create an Ansible Controller and Managed Nodes
* Configure one Ubuntu Controller and two Managed Nodes
* Test Ansible connectivity using SSH
* Execute ad-hoc commands from the controller

## Lab Architecture

| Server     | OS                       | Role                 |
| ---------- | ------------------------ | -------------------- |
| Controller | Ubuntu                   | Ansible Control Node |
| Node-1     | Red Hat Enterprise Linux | Managed Node         |
| Node-2     | Amazon Linux 2023        | Managed Node         |

---

## Install Ansible

### Controller (Ubuntu)

```bash
sudo apt update
sudo apt install ansible -y
```

### Verify Installation

```bash
ansible --version
```

---

## Create Ansible User

### Node-1 (RHEL)

```bash
sudo useradd ansible
sudo passwd ansible
```

### Node-2 (Amazon Linux 2023)

```bash
sudo useradd ansible
sudo passwd ansible
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

## Create Inventory

Example inventory:

```ini
[node]
172.31.xx.xx
172.31.xx.xx

[node:vars]
ansible_user=ansible
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

