# Lab 008: Install Jenkins Using Ansible

## Lab goal

In this lab, we will learn how to use Ansible to install Jenkins.

## Environments

| #  | Env  | Y/N  | Recommended   |  Comment |
|---|---|---|---|---|
| 1 | Windows only | YN | YN |   |
| 2 | Windows + Ubuntu | Y | Y |   |
| 3 | Mac only | YN | YN |   |
| 4 | Mac + Ubuntu | Y | Y |   |

This lab has to use Ubuntu (and docker inside it).

[With_Windows_Ubuntu](02_Y_Windows_Ubuntu.md)

<!--
[Windows Only](01_N_WindowsOnly.md)

[With_Windows_Ubuntu](02_Y_Windows_Ubuntu.md)

[Mac Only](03_N_MacOnly.md)

[With_Mac_Ubuntu](04_Y_MacDocker_Ubuntu.md)

[With_Mac_Ubuntu](04_Y_Mac_UbuntuDocker.md)
-->

<!--
# <a name="troubleshooting">Troubleshooting</a>

## Issue 1: The IP address configured for the host-only network is not within the

allowed ranges.
When running `vagrant up`, showing below error:

```
The IP address configured for the host-only network is not within the
allowed ranges. Please update the address used to be within the allowed
ranges and run the command again.

  Address: 192.168.33.10
  Ranges: 192.168.56.0/21

Valid ranges can be modified in the /etc/vbox/networks.conf file. For
more information including valid format see:

  https://www.virtualbox.org/manual/ch06.html#network_hostonly
```

**Solution:**
ref: <https://stackoverflow.com/questions/70704093/the-ip-address-configured-for-the-host-only-network-is-not-within-the-allowed-ra>

# <a name="reference">Reference</a>

[Install Jenkins in Linux](https://www.jenkins.io/doc/book/installing/linux/)</br>
[Install Jenkins Using Ansible On Ubuntu](https://blog.knoldus.com/how-to-install-jenkins-using-ansible-on-ubuntu/)</br>
[Installing Jenkins using an Ansible Playbook](https://medium.com/nerd-for-tech/installing-jenkins-using-an-ansible-playbook-2d99303a235f)</br>
[Jenkins Role](https://galaxy.ansible.com/geerlingguy/jenkins)</br>
-->
