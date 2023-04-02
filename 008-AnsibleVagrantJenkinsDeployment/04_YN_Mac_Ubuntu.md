# Project 008: Install Jenkins Using Ansible

## Project Goal

In this lab, we will learn how to use Ansible to install Jenkins.

## Prerequisites

### 1. Install Docker for Mac

### 2. Install Vagrant for Mac

## Steps

### 1. Create a VM to Install Jenkins

We are using **Vagrant** to create a VM to install the Jenkins application

```bash
vagrant provision
vagrant up
```

> Note: Select the network card which is being used to connect to Internet

### 2. Run Ansible Playbook

> Note: Before we run the Ansible Playbook, we need to SSH into the Vagrant VM created above and accept the finger print. If we don't do this, then we may encounter errors when we try and run the Ansible Playbook

```bash
ssh-copy-id admin@192.168.33.10
ssh admin@192.168.33.10 
exit
```

We can run below **ad-hoc** command to make sure the Ansible is able to talk to the VM:

```bash
ansible -i hosts.ini jenkins_vm -m ping 
```

> Note: If we are using other VM instead of Vagrant, we need to update the IP in `hosts.ini`
we should get below response if it is successful

```bash
jenkins_vm | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

Then, in our **local host**, run below Ansible playbook script:

```bash
ansible-playbook install-jenkins.yml -i hosts.ini --ask-pass --ask-become-pass
```

> Note: The password is stored in `Vagrantfile` for `admin` user if we are using Vagrant as VM. The default is `admin123`. we should see below output if the installation is successful.

Below output when the deployment is done:

```bash
SSH password: 
BECOME password[defaults to SSH password]: 

PLAY [jenkins_vm] **************************************************************************************
...
PLAY RECAP *********************************************************************************************
jenkins_vm                 : ok=6    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```

### 3. Unlocking Jenkins

When we first access the new Jenkins install, we are asked to unlock it using an automatically-generated password.

a. **Browse** to <http://192.168.33.10> (or whichever port we configured for Jenkins when installing it) and wait until the Unlock Jenkins page appears.

b. Run `sudo cat /var/lib/jenkins/secrets/initialAdminPassword` in the Vagrant VM or any other VM we installed the Jenkins and enter the password showed and click "Next".

c. Click **"Install suggested plugins"** and wait for all plugins are installed.

d. Fill out the info for our First **Amdin User**.

- **Username:** admin
- **Password:** test123  
- **Confirm password:** test123
- **Full name:** Jenkins
- **E-mail address:** jenkins@gmail.com

Click **"Save and Continue"**

e. Click **"Save and Finish"** and click **"Start using Jenkins"**. Then we should login as the admin user we just created previously

### 4. Using Ansibel Role

we are **done** with the Jenkins via Ansible.

Now, we are going to use **Ansible Role** to install the Jenkins instead. **Ansible Role** are consists of many playbooks and it is a way to group multiple tasks together into one container to do automation in very effective manner with clean directory structures. It can be easily reuse the codes by anyone if it it is suitable.

Before that, we can uninstall the Jenkins/Java package in the Vagrant VM, if we are going to use the same VM.

We are going to apply the Ansible Playbook `uninstall-jenkins.yaml` to remove the related packages before we start the new deployment:

```bash
ansible-playbook uninstall-jenkins.yml -i hosts.ini --ask-pass --ask-become-pass
```

Run below command to download the Jenkins Role from **Ansible Galaxy**:

```bash
ansible-galaxy install geerlingguy.jenkins
```

The Role will be installed under `~/.ansible/roles`

```bash
cd ~/.ansible/roles/geerlingguy.jenkins
ls
defaults  handlers  LICENSE  meta  molecule  README.md  tasks  templates  tests  vars
```

Roles expect files to be in certain directory names. Each directory must contain a `main.yml` file. Below is a describption of each directory.

- **tasks** - Contains the main list of tasks to be executed by the role
- **handlers** - contains handlers, which may be used by this role or even anywhere outside this role
- **defaults** - default variables for the role
- **vars** - other variables for the role.
- **files** - containers file which can be deployed via this role
- **templates** - contains templates which can be deployed via this role.
- **meta** - defines some meta data for this role.

Run below command to apply the **Ansible Role**:

```bash
ansible-playbook install-jenkins-role.yml -i hosts.ini --ask-pass --ask-become-pass
```
