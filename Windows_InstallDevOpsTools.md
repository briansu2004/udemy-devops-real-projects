# How to set up Windows

## Install Docker (and Docker Compose) in Windows

<https://docs.docker.com/desktop/install/windows-install/>

## Install VirtualBox in Windows

<https://www.virtualbox.org/wiki/Downloads>

## Install Vagrant in Windows

<https://developer.hashicorp.com/vagrant/docs/installation>

### Use Docker as the provider for Vagrant on Windows

To use Docker as the provider for Vagrant on our Windows, follow these steps:

Install Vagrant on our Windows by following the instructions on the official Vagrant documentation.

Install the Docker provider plugin for Vagrant by running the following command in our Terminal window:

`vagrant plugin install vagrant-docker-compose`

This command will install the Docker provider plugin for Vagrant.

Create a new Vagrantfile in our project directory or use an existing one.

In our Vagrantfile, configure Vagrant to use the Docker provider by adding the following line:

`config.vm.provider "docker"`

You can also configure additional settings for the Docker provider, such as the image to use, by adding more lines to our Vagrantfile.

Run the vagrant up command to start the Vagrant environment with the Docker provider.

`vagrant up --provider=docker`

This command will start the Vagrant environment with the Docker provider.

Once the Vagrant environment is started, we can use the vagrant ssh command to SSH into the environment and interact with our Docker containers.

## Install minikube in Windows

<https://minikube.sigs.k8s.io/docs/start/>

```dos
minikube start --driver=docker --kubernetes-version=v1.26.1

minikube version
```

## Install kubectl in Windows

```dos
minikube kubectl

kubectl version
```

## Install Helm in Windows

```dos
choco install kubernetes-helm

helm version
```

## Install Kind in Windows

```bash
choco install kind
```

## Install Git Bash in Windows

Here are the steps to install Git Bash on Windows:

Download the Git for Windows installer from the official website at <https://gitforwindows.org/>.

Run the installer and follow the prompts. When prompted for installation options, keep the default settings and click "Next" to proceed.

On the "Adjusting your PATH environment" screen, select "Use Git and optional Unix tools from the Command Prompt" and click "Next".

On the "Configuring the line ending conversions" screen, select "Checkout Windows-style, commit Unix-style line endings" and click "Next".

On the "Configuring the terminal emulator to use with Git Bash" screen, select "Use Windows' default console window" and click "Next".

On the "Configuring extra options" screen, leave the default settings and click "Next".

On the "Installing" screen, click "Install" to begin the installation process.

Once the installation is complete, click "Finish" to exit the installer.

You can now launch Git Bash by searching for "Git Bash" in the Start menu or by clicking the Git Bash shortcut on your desktop.

Git Bash is now installed on your Windows system, and you can use it to run Git commands and Unix utilities from the command line.
