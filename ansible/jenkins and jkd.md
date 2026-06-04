# Jenkins Installation using Ansible on openSUSE
Objective:

Install Java and Jenkins on an openSUSE managed node using Ansible.

This lab helped me understand that before writing an Ansible playbook, it is important to know how to perform the installation manually. Once the manual process is clear, converting it into Ansible tasks becomes much easier.

# my understanding
Initially, I tried creating the playbook directly by searching for Ansible modules and examples online.

This led to multiple issues:

* Wrong repository format
*  Wrong GPG key URL
*  Unsupported Ansible module parameters
*  Mixing Ubuntu repository configuration with openSUSE

I realized that the better approach is:

* Install the software manually.
* Understand every command.
* Verify the installation works.
* Convert the commands into Ansible tasks.

This makes troubleshooting much easier.

Instead of searching:

"How to write Ansible playbook for Jenkins?"

I searched:

"How to install Jenkins on openSUSE"

The installation instructions revealed:

* Repository URL
* GPG key location
* Package name
* Service name

These values became the foundation of the playbook.

# Finding that GPG key

To understand where the GPG key comes from:
```
wget https://pkg.jenkins.io/rpm-stable/jenkins.repo

cat jenkins.repo
```
the output was like this
```
[ansible@ip-172-31-35-187 ansible]$ cat jenkins.repo
[jenkins]
name=Jenkins-stable
enabled=1
type=rpm-md
baseurl=https://pkg.jenkins.io/rpm-stable
gpgkey=https://pkg.jenkins.io/rpm-stable/repodata/repomd.xml.key
gpgcheck=1
repo_gpgcheck=1

# Only for SUSE/openSUSE distributions with zypper
autorefresh=1
keeppackages=0
```
This repository file provides:

* Base Repository URL
* GPG Key URL
* Repository Name

This information helped me. it is then used in Ansible playbook bulding.

---------------------

# playbook
```
---
- name: Install Java and Jenkins
  hosts: node1
  become: yes

  tasks:

    - name: Install Java
      community.general.zypper:
        name: java-21-openjdk
        state: present

    - name: Download Jenkins Repository File
      ansible.builtin.get_url:
        url: https://pkg.jenkins.io/rpm-stable/jenkins.repo
        dest: /etc/zypp/repos.d/jenkins.repo
        mode: '0644'

    - name: Refresh Repository Cache
      ansible.builtin.command: zypper refresh

    - name: Install Jenkins
      community.general.zypper:
        name: jenkins
        state: present

    - name: Start Jenkins Service
      ansible.builtin.systemd:
        name: jenkins
        state: started
        enabled: yes

    - name: Verify Jenkins Status
      ansible.builtin.command: systemctl is-active jenkins
      register: jenkins_status
      changed_when: false

    - name: Display Jenkins Status
      ansible.builtin.debug:
        var: jenkins_status.stdout

````
Remember: Start with the manual installation process.

Once the manual steps are understood, Ansible becomes a way to automate those exact steps rather than a tool for guessing how the installation works.
