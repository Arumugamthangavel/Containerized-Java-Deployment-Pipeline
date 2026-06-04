# Docker and Tomcat Installation using Ansible on Ubuntu

## Objective

Install Docker and Apache Tomcat on an Ubuntu managed node using Ansible.

This lab helped me understand the difference between package management tasks and service management tasks in Ansible.

It also reinforced an important lesson:

Before writing an Ansible playbook, understand the manual installation process first.

Once the manual installation works, converting those steps into Ansible tasks becomes straightforward.

# my understanding

Initially, I wrote the playbook directly without verifying the installation manually.

This caused several issues:

* Using the `apt` module to start services
* Missing Java dependency for Tomcat
* Using incorrect Tomcat package names
* Confusion between package names and service names

I realized that Ansible modules have specific purposes.

For example:

* `apt` → Install or remove packages
* `service` or `systemd` → Start, stop, or enable services

Trying to manage services using the `apt` module resulted in errors.

The better approach is:

* Install the software manually
* Understand package names
* Identify service names
* Verify everything works
* Convert the steps into Ansible tasks

This makes troubleshooting much easier.

Instead of searching:

"How to write Ansible playbook for Docker and Tomcat?"

I searched:

"How to install Docker on Ubuntu"

and

"How to install Tomcat on Ubuntu"

The installation instructions revealed:

* Docker package name
* Java dependency
* Tomcat package name
* Service names

These values became the foundation of the playbook.

---

# Finding Package Names

To verify available Tomcat packages:

```bash
apt search tomcat
```

This helped identify the correct package names available on Ubuntu.

Example output included:

```text
tomcat10
tomcat10-admin
```

Similarly, Docker was available through:

```bash
apt search docker.io
```

Output:

```text
docker.io
```

This information was then used in the playbook.

---

# Finding Service Names

Installing a package does not always reveal the service name.

To verify service names:

```bash
systemctl list-unit-files | grep docker
```

Output:

```text
docker.service
```

For Tomcat:

```bash
systemctl list-unit-files | grep tomcat
```

Output:

```text
tomcat10.service
```

These service names were required when creating the service tasks.

---

# playbook

```yaml
---
- name: Install Docker and Tomcat on Node-2
  hosts: node-2
  become: yes

  tasks:

    - name: Update apt package index
      apt:
        update_cache: yes

    - name: Install Docker
      apt:
        name: docker.io
        state: present

    - name: Start and enable Docker
      service:
        name: docker
        state: started
        enabled: yes

    - name: Install Java
      apt:
        name: openjdk-17-jdk
        state: present

    - name: Install Tomcat
      apt:
        name:
          - tomcat10
          - tomcat10-admin
        state: present

    - name: Start and enable Tomcat
      service:
        name: tomcat10
        state: started
        enabled: yes
```

---

# Verification

Verify Docker:

```bash
docker --version
```

Verify Docker service:

```bash
systemctl status docker
```

Verify Java:

```bash
java -version
```

Verify Tomcat:

```bash
systemctl status tomcat10
```

Verify Tomcat web page:

```text
http://<server-ip>:8080
```

If the Tomcat welcome page appears, the installation is successful.

---

# Lesson Learned

One of the biggest lessons from this lab was understanding the difference between package installation and service management.

My initial mistake was:

```yaml
apt:
  name: docker.io
  state: started
```

The `apt` module cannot start services.

The correct approach is:

```yaml
service:
  name: docker
  state: started
```

Understanding the purpose of each module makes writing Ansible playbooks much easier.

Remember: Start with the manual installation process.

Once the manual steps are understood, Ansible becomes a way to automate those exact steps rather than a tool for guessing how the installation works.
