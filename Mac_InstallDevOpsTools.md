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
