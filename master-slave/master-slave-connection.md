# Jenkins Master and Agent Configuration

## Objective

Configure Jenkins Master on Node-1 and connect Node-2 as a Jenkins Agent using SSH.

The goal of this lab was to understand how Jenkins distributes workloads across multiple servers and executes jobs remotely using agents.

* Jenkins Master connected to Agent
* Agent appears online in Jenkins
* Jobs execute on Node-2 instead of Node-1

---

# my understanding

Initially, I thought adding a node in Jenkins would automatically make it a remote agent.

My first attempt was adding the Jenkins Master machine itself as an agent using:

```text
Launch agent by connecting it to the controller
```

The node became online successfully.

However, after testing, I realized that Jenkins was simply running jobs on the same machine where Jenkins was installed.

This taught me the difference between:

* Controller node
* Agent node
* SSH-based agents
* Inbound agents

The actual lab requirement was:

```text
Node-1 = Master
Node-2 = Agent
```

using an SSH connection.

---

# Preparing Node-2

A dedicated user was created for Jenkins communication.

```bash
sudo useradd -m -s /bin/bash ansible
sudo passwd ansible
```

Java was already installed because it is required for Jenkins Remoting.

Verification:

```bash
java -version
```

---

# Creating SSH Authentication

On Node-1, Jenkins uses SSH keys to access Node-2.

Switch to Jenkins user:

```bash
sudo su - jenkins
```

Generate key pair:

```bash
ssh-keygen
```

View public key:

```bash
cat ~/.ssh/id_rsa.pub
```

---

# Configure Authorized Keys on Node-2

Switch to ansible user:

```bash
sudo su - ansible
```

Create SSH directory:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
```

Create authorized_keys:

```bash
vi ~/.ssh/authorized_keys
```

Paste the Jenkins public key.

Set permissions:

```bash
chmod 600 ~/.ssh/authorized_keys
```

---

# Verify SSH Connectivity

Before configuring Jenkins, test manually.

From Node-1:

```bash
sudo su - jenkins
ssh ansible@172.31.43.158
```

Successful login confirms:

* SSH service running
* Security groups configured correctly
* Key authentication working

---

# Jenkins Agent Configuration

Navigate to:

```text
Manage Jenkins
→ Nodes
→ New Node
```

Node Name:

```text
node-2
```

Node Type:

```text
Permanent Agent
```

---

## Agent Settings

Executors:

```text
2
```

Remote Root Directory:

```text
/home/ansible
```

Labels:

```text
ubuntu
```

Launch Method:

```text
Launch agents via SSH
```

Host:

```text
172.31.43.158
```

Credentials:

```text
SSH Username with Private Key
```

Username:

```text
ansible
```

Private Key:

```text
Contents of Jenkins user's id_rsa
```

Host Key Verification:

```text
Non verifying Verification Strategy
```

---

# First Error Encountered

Initial connection failed with:

```text
Server rejected the 1 private key(s)
Authentication failed
```

This indicated:

* Jenkins could reach Node-2
* SSH port was open
* Authentication was failing

The issue was fixed by correctly configuring:

```text
authorized_keys
```

for the ansible user on Node-2.

After updating the SSH key configuration:

```text
Authentication successful
```

appeared in Jenkins logs.

---

# Second Error Encountered

After SSH authentication succeeded, Jenkins failed during agent startup.

Error:

```text
UnsupportedClassVersionError
```

Detailed message:

```text
class file version 65.0
this version of the Java Runtime only recognizes class file versions up to 61.0
```

---

# Understanding the Problem

Java Class Versions:

| Class Version | Java Version |
| ------------- | ------------ |
| 61            | Java 17      |
| 65            | Java 21      |

Node-1:

```text
Java 21
```

Node-2:

```text
Java 17
```

Jenkins Remoting was compiled for Java 21 but Node-2 was attempting to run it using Java 17.

---

# Additional Issue

While troubleshooting, Jenkins agent extraction required additional temporary storage.

Checking disk usage:

```bash
df -h
```

showed the temporary filesystem was too small.

The size was increased:

```bash
sudo mount -o remount,size=5G /tmp
```

Verify:

```bash
df -h
```

This provided sufficient temporary space for Jenkins operations.

---

# Fix

Install Java 21 on Node-2.

```bash
sudo apt update
sudo apt install openjdk-21-jdk -y
```

Verify:

```bash
java -version
```

Expected:

```text
openjdk version "21"
```

---

# Successful Connection

After upgrading Java:

```text
Authentication successful
```

Agent startup completed successfully.

Node status changed to:

```text
Connected
```

inside Jenkins.

---

# Testing Job Execution

Created a Freestyle Project.

Restricted execution to:

```text
ubuntu
```

Build step:

```bash
hostname
whoami
pwd
```

Expected output:

```text
ip-172-31-43-158
ansible
/home/ansible
```

This confirmed the job was executing on Node-2 instead of the Jenkins Master.



Understanding the logs made troubleshooting much easier.
