# How to set up Mac

## Install Homebrew (and Git) in Mac

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

## Install Vagrant in Mac

```bash
brew install vagrant

vagrant --version
```

### Use Docker as the provider for Vagrant on Mac

To use Docker as the provider for Vagrant on our Mac, follow these steps:

Install Vagrant on our Mac by following the instructions on the official Vagrant documentation.

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

## Install VirtualBox in Mac

```bash
brew install VirtualBox

VBoxManage --version
```

## Install Docker (and Docker Compose) in Mac

Docker Desktop

```bash
brew install docker

docker version
```

## Install minikube in Mac

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64 && \
sudo install minikube-darwin-amd64 /usr/local/bin/minikube

minikube version
```

## Install kubectl in Mac

```bash
brew install kubectl

kubectl version
```

## Install Helm in Mac

```bash
brew install helm

helm version
```

## Install Kind in Mac

```bash
brew install kind
```
