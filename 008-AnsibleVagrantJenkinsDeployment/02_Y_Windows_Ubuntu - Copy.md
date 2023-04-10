# Lab 008: Install Jenkins Using Ansible

Windows + Ubuntu

<!--
Still have the same issue.

Solutions:

1. Try again at home

2. Wait for tomorrow

Last Sunday also had same experience!

3. make sure jenkins is available for the command `apt search jenkins`!

Maybe Sunday they are doing some updates!

```bash
vagrant@vagrant:~$ sudo apt-get install jenkins
Reading package lists... Done
Building dependency tree       
Reading state information... Done
Package jenkins is not available, but is referred to by another package.
This may mean that the package is missing, has been obsoleted, or
is only available from another source

E: Package 'jenkins' has no installation candidate

vagrant@vagrant:~$ apt search jenkins | grep jenkins

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

jenkins-debian-glue/focal,focal 0.20.1 all
jenkins-debian-glue-buildenv/focal,focal 0.20.1 all
jenkins-job-builder/focal,focal 3.2.0-1 all
jenkins-job-builder-doc/focal,focal 3.2.0-1 all
libjenkins-htmlunit-core-js-java/focal,focal 2.6-hudson-1-1fakesync1 all
libjenkins-json-java/focal,focal 2.4-jenkins-3-5 all
libjenkins-json-java-doc/focal,focal 2.4-jenkins-3-5 all
  Documentation for libjenkins-json-java
libjenkins-trilead-ssh2-java/focal,focal 217-jenkins-8-1 all
libjenkins-trilead-ssh2-java-doc/focal,focal 217-jenkins-8-1 all
  Documentation for libjenkins-trilead-ssh2-java
python-jenkins-doc/focal,focal 0.4.16-1 all
python3-jenkins/focal,focal 0.4.16-1 all
python3-jenkins-job-builder/focal,focal 3.2.0-1 all
python3-jenkinsapi/focal,focal 0.3.11-1ubuntu1 all
```

Btw <https://pkg.origin.jenkins.io/debian-stable/> has the official information.
-->

<!--
Issues:

Can't use a Windows system for the Ansible control node.

Install 2 Vagrant VMs in Windows?
-->

## Lab goal

In this lab, we will learn how to use Ansible to install Jenkins.

## Prerequisites

### 1. Install 2 Vagrant VMs for Windows

Because we can't install ansible for Windows directly.

1 Vagrant VM will be installed ansible.

### 2. Install Ansible for Ubuntu

```bash
sudo apt update
sudo apt upgrade
sudo apt install ansible -y
ansible --version
```

<!--
```bash
vagrant@vagrant:~$ ansible --version

ansible 2.9.6
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/vagrant/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.8.10 (default, Mar 13 2023, 10:26:41) [GCC 9.4.0]
```
-->

## Steps

### 1. Run Ansible Playbook

> Note: Before we run the Ansible Playbook, we need to SSH into the Vagrant VM created above and accept the finger print. If we don't do this, then we may encounter errors when we try and run the Ansible Playbook

```bash
ssh-keygen -t rsa -b 4096

ssh-copy-id admin@192.168.33.10

ssh admin@192.168.33.10 
exit
```

<!--
```bash
DevOps 🚀 devbox % ssh-copy-id admin@192.168.33.10
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/Users/x239757/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if we are prompted now it is to install the new keys
admin@192.168.33.10's password: 
Permission denied, please try again.
admin@192.168.33.10's password: 

Number of key(s) added:        1

Now try logging into the machine, with:   "ssh 'admin@192.168.33.10'"
and check to make sure that only the key(s) we wanted were added.

DevOps 🚀 devbox % ssh admin@192.168.33.10 
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.4.0-42-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
New release '22.04.2 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

Last login: Sun Apr  2 15:29:13 2023 from 192.168.33.1
$ exit
Connection to 192.168.33.10 closed.
DevOps 🚀 devbox % 
```
-->

We can run below **ad-hoc** command to make sure the Ansible is able to talk to the VM:

```bash
git clone https://github.com/briansu2004/udemy-devops-real-projects.git
cd udemy-devops-real-projects/008-AnsibleVagrantJenkinsDeployment
ansible -i hosts.ini jenkins_vm -m ping 
```

<!--
> Note: If we are using other VM instead of Vagrant, we need to update the IP in `hosts.ini`
-->

We should get below response if it is successful.

```bash
jenkins_vm | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

Run below Ansible playbook script:

```bash
sudo apt-get install sshpass

ansible-playbook install-jenkins.yml -i hosts.ini --ask-pass --ask-become-pass
```

<!--
> Note: The password is stored in `Vagrantfile` for `admin` user if we are using Vagrant as VM. The default is `admin123`. We should see below output if the installation is successful.
-->

Below output when the deployment is done:

```bash
SSH password: 
BECOME password[defaults to SSH password]: 

PLAY [jenkins_vm] **************************************************************************************
...
PLAY RECAP *********************************************************************************************
jenkins_vm                 : ok=6    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```

<!--
`dpkg --get-selections | grep jenkins`

Before -

```bash
vagrant@vagrant:~$ dpkg --get-selections | grep jenkins
jenkins                                         deinstall
```

After -

```bash
vagrant@vagrant:~$ dpkg --get-selections | grep jenkins
jenkins                                         install
```
-->

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

We are **done** with the Jenkins via Ansible.

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
