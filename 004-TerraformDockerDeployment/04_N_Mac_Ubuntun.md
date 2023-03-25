# Project 004: Deploy Docker with Terraform Script

Mac + Ubuntu

Issues:

There are issues to install Terraform in Ubuntu (in Mac).

But no issues to nstall Terraform in Ubuntu (in Windows)!

<!--
## <a name="prerequisites">Prerequisites</a>

- Ubuntu 20.04 OS (Minimum 2 core CPU/8GB RAM/30GB Disk)
- Docker (see installation guide [here](https://docs.docker.com/get-docker/))
- Docker Compose (see installation guide [here](https://docs.docker.com/compose/install/))
- Terraform (see installation guide [here](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli))
-->

## Prerequisites

### 1. Install and start Vagrant

```bash
vagrant up 
vagrant ssh
```

### 2. Install Docker and Docker Compose in Vagrant

```bash
cat > install_docker.sh <<EOF

sudo apt-get update
echo y | sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
sudo mkdir -p /etc/apt/keyrings
sudo rm -f /etc/apt/keyrings/docker.gpg
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo chmod a+r /etc/apt/keyrings/docker.gpg
sudo apt-get update
echo y | sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
sudo chmod 666 /var/run/docker.sock
EOF

chmod 777 install_docker.sh
./install_docker.sh

sudo apt install docker-compose
```

## Steps

## 1. Config the GitLab domain name

In this lab, we will use `mydevopsrealprojects.com` as the GitLab domain name.

Hence the gitlab URL is `http://gitlab.mydevopsrealprojects.com`

We need it in the `docker-compose.yml` file.

## 2. Configure the **hosts** file

<!--
Windows: `C:\Windows\System32\drivers\etc\hosts`

In local Windows's hosts file `C:\Windows\System32\drivers\etc\hosts`

```dos
192.168.33.10 gitlab.mydevopsrealprojects.com
192.168.33.10 registry.gitlab.mydevopsrealprojects.com
```

Unix / Mac: `/etc/hosts`

In Vagrant Ubuntu's hosts file `/etc/hosts`

```bash
sudo vi /etc/hosts
```
-->

Add these 2 entries in Vagrant Ubuntu's hosts file `/etc/hosts` -

```bash
127.0.0.1 gitlab.mydevopsrealprojects.com
127.0.0.1 registry.gitlab.mydevopsrealprojects.com
```

<!--
## 3. Config the root password

```yml
    hostname: 'gitlab.mydevopsrealprojects.com'
    environment:
      GITLAB_ROOT_PASSWORD: "Password2023#"
      EXTERNAL_URL: "http://gitlab.mydevopsrealprojects.com"
      GITLAB_OMNIBUS_CONFIG: |
        gitlab_rails['initial_root_password'] = "Password2023#"
        gitlab_rails['store_initial_root_password'] = true
        gitlab_rails['display_initial_root_password'] = true
```
-->

## 3. Docker compose

```bash
git clone https://github.com/briansu2004/udemy-devops-real-projects.git
cd udemy-devops-real-projects/004-TerraformDockerDeployment
docker-compose up
```

<!--
### 1. Deploy a gitlab server to store the terraform state file

```bash
git clone https://github.com/briansu2004/udemy-devops-real-projects.git
cd udemy-devops-real-projects/004-TerraformDockerDeployment
docker-compose up
```

> Note: Once the gitlab container is fully up running, you can run below command to retrieve the initial password, if you haven't specified it in the deployment file. The default admin username should be `root`

```bash
sudo docker exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password
```

### 2. Add the new DNS record in your local hosts file

In your `docker-compose.yaml`, you have defined your gitlab server hostname in `hostname` field. Add it to your local hosts file so that you can use it to git clone the repo from your gitlab server.

```bash
export Your_Local_Host_IP=<Your_Local_Host_IP>
echo "${Your_Local_Host_IP}  gitlab.mydevopsrealprojects.com" | sudo tee -a /etc/hosts
```

e.g.

```bash
echo "${Your_Local_Host_IP}  gitlab.mydevopsrealprojects.com" | sudo tee -a /etc/hosts
```

Then you should be able to access the Gitlab website via `https://mydevopsrealprojects.com`
-->

<!--
### 4. Update the Gitlab original Certificate

Since the initial Gitlab server **certificate** is missing some info, you may have to **regenerate** a new one and **reconfigure** in the gitlab server. Run below commands:

```bash
docker exec -it gitlab bash
mkdir /etc/gitlab/ssl_backup
mv /etc/gitlab/ssl/* /etc/gitlab/ssl_backup
cd /etc/gitlab/ssl
openssl genrsa -out ca.key 2048
openssl req -new -x509 -days 365 -key ca.key -subj "/C=CN/ST=GD/L=SZ/O=Acme, Inc./CN=Acme Root CA" -out ca.crt

# Note: Make sure to replace below `YOUR_GITLAB_DOMAIN` with your own domain name. For example, mydevopsrealprojects.com.
# Certificate for gitlab server
export YOUR_GITLAB_DOMAIN=mydevopsrealprojects.com
openssl req -newkey rsa:2048 -nodes -keyout gitlab.$YOUR_GITLAB_DOMAIN.key -subj "/C=CN/ST=GD/L=SZ/O=Acme, Inc./CN=*.$YOUR_GITLAB_DOMAIN" -out gitlab.$YOUR_GITLAB_DOMAIN.csr
openssl x509 -req -extfile <(printf "subjectAltName=DNS:$YOUR_GITLAB_DOMAIN,DNS:gitlab.$YOUR_GITLAB_DOMAIN") -days 365 -in gitlab.$YOUR_GITLAB_DOMAIN.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out gitlab.$YOUR_GITLAB_DOMAIN.crt

# Certificate for container registry
openssl req -newkey rsa:2048 -nodes -keyout registry.gitlab.$YOUR_GITLAB_DOMAIN.key -subj "/C=CN/ST=GD/L=SZ/O=Acme, Inc./CN=*.$YOUR_GITLAB_DOMAIN" -out registry.gitlab.$YOUR_GITLAB_DOMAIN.csr
openssl x509 -req -extfile <(printf "subjectAltName=DNS:$YOUR_GITLAB_DOMAIN,DNS:gitlab.$YOUR_GITLAB_DOMAIN,DNS:registry.gitlab.$YOUR_GITLAB_DOMAIN") -days 365 -in registry.gitlab.$YOUR_GITLAB_DOMAIN.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out registry.gitlab.$YOUR_GITLAB_DOMAIN.crt
gitlab-ctl reconfigure
gitlab-ctl restart
exit
```

### 4. Import the gitlab new certificate in your local host CA chains

In order to make your local host be able to talk to the gitlab server via TLS, you have to import the new gitlab certificate, which is generated previous step, into your local host CA store chains. Login to your local host and run below command:

```bash
export YOUR_GITLAB_DOMAIN=mydevopsrealprojects.com
sudo docker cp gitlab:/etc/gitlab/ssl/gitlab.$YOUR_GITLAB_DOMAIN.crt /usr/local/share/ca-certificates/
sudo update-ca-certificates
```

> Note: If you are using CentOS, you may need to include "-addext basicConfstraints=critical,CA:TRUE" in the ca.crt file and use `update-ca-trust` command instead.

```bash
# For CentOS
openssl req -new -x509 -days 365 -key ca.key -addext basicConstraints=critical,CA:TRUE -subj "/C=CN/ST=GD/L=SZ/0=Acme, Inc./CN=Acme Root CA"  -out ca.crt
```
-->

### 5. Create a new project in your Gitlab server and generate a personal access token

Login to your Gitlab server website (`https://mydevopsrealprojects.com`) and Click **"New project"** -> **"Create blank project"** -> Type a project name in **"Project name"**, i.g. *first_project*, select **"Public"** in **Visiblity Level** section -> Click **"Create project"** </br>

Once the project is create, go to **"Setting""** -> **"Access Tokens"** -> Type a customized token name in **Token name** field, i.ig  *terraform-token* , Select a role **"Maintainer"** in *Select a role field*, Select scopes **"api/read_api/read_repositry/write_repository"** </br>

Make a note of the new token generated as you will need to apply it in the next step.

<!-- glpat-y7Rs81efD5hSVZxX_TZ3 -->

![gitlab-personal-accees-token](images/gitlab-personal-accees-token.png)

### 6. Update `config.tfbackend`

Before running `terraform init`, you have to update `config/test/config.tfbackend` file with the credential/gitlab server info accordingly. The below is the definition for the variables:</br>

- **PROJECT_ID:** Go to the project and head to **"Setting"** -> **"General"**, and you will see **"Project ID"** in the page. </br>
- **TF_USERNAME:** If you haven't created your own user, the default user should be `root` </br>
- **TF_PASSWORD:** This is the gitlab **personal access token**, which you can fetch from previous step </br>
- **TF_ADDRESS:** This is URL to store your **terraform state file**.
  The pattern is like `https://<your gitlab server url>/api/v4/projects/<your project id>/terraform/state/old-state-name`.
  For example: `https://gitlab.mydevopsrealprojects.com/api/v4/projects/${PROJECT_ID}/terraform/state/old-state-name`

<!--
```bash
docker exec -it gitlab bash
cd ~/udemy-devops-real-projects/004-TerraformDockerDeployment
sudo vim config/test/config.tfbackend
```
-->

### 7. Run terraform script to deploy the infra

Init

```bash
terraform init -backend-config=config/test/config.tfbackend
```

Plan

```bash
terraform plan -var-file=config/test/test.tfvars -out deploy.tfplan
```

Apply

```bash
terraform apply deploy.tfplan
```

### 8. Verification

You should be able to visit the website via `http://0.0.0.0:8080`

![hello-world](images/hello-world.png)

## Clean up

Delete the terraform infra

```bash
terraform destroy -var-file=config/test/test.tfvars 
```

Delete the gitlab container, as well as the volumes mounted

```bash
docker-compose down -v
```

Delete the test container if exists

```bash
docker rm -f terraform-docker-example
```
