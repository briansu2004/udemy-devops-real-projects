# Project 004: Deploy Docker with Terraform Script

Windows only

<!--
(No issues to install Terraform in Ubuntu.)

Issues:

terraform init -backend-config=config/test/config.tfbackend has issues.

```dos
C:\devbox\udemy-devops-real-projects\004-TerraformDockerDeployment>terraform init -backend-config=config/test/config.tfbackend

Initializing the backend...
2023/03/25 17:33:48 [DEBUG] GET https://gitlab.mydevopsrealprojects.com/api/v4/projects/2/terraform/state/old-state-name
2023/03/25 17:33:49 [ERR] GET https://gitlab.mydevopsrealprojects.com/api/v4/projects/2/terraform/state/old-state-name request failed: Get "https://gitlab.mydevopsrealprojects.com/api/v4/projects/2/terraform/state/old-state-name": EOF
2023/03/25 17:33:49 [DEBUG] GET https://gitlab.mydevopsrealprojects.com/api/v4/projects/2/terraform/state/old-state-name: retrying in 5s (2 left)
2023/03/25 17:33:54 [ERR] GET https://gitlab.mydevopsrealprojects.com/api/v4/projects/2/terraform/state/old-state-name request failed: Get "https://gitlab.mydevopsrealprojects.com/api/v4/projects/2/terraform/state/old-state-name": EOF
2023/03/25 17:33:54 [DEBUG] GET https://gitlab.mydevopsrealprojects.com/api/v4/projects/2/terraform/state/old-state-name: retrying in 10s (1 left)
2023/03/25 17:34:04 [ERR] GET https://gitlab.mydevopsrealprojects.com/api/v4/projects/2/terraform/state/old-state-name request failed: Get "https://gitlab.mydevopsrealprojects.com/api/v4/projects/2/terraform/state/old-state-name": EOF
Error refreshing state: Failed to get state: GET https://gitlab.mydevopsrealprojects.com/api/v4/projects/2/terraform/state/old-state-name giving up after 3 attempts
```

Root cause:

Need to configure the certificate.

And can't do the same steps for Windows.
-->

## Prerequisites

### 1. Install and start Docker for Windows

Download and install

### 2. Install Terraform for Windows

```dos
choco install terraform
```

## Steps

## 1. Config the GitLab domain name

In this lab, we will use `mydevopsrealprojects.com` as the GitLab domain name.

Hence the GitLab server URL is `http://gitlab.mydevopsrealprojects.com`

## 2. Configure the **hosts** file

Add these 2 entries in the Windows hosts file `C:\Windows\System32\drivers\etc\hosts`

```dos
127.0.0.1 gitlab.mydevopsrealprojects.com
127.0.0.1 registry.gitlab.mydevopsrealprojects.com
```

## 3. Start the docker compose

```dos
git clone https://github.com/briansu2004/udemy-devops-real-projects.git
cd udemy-devops-real-projects/004-TerraformDockerDeployment
docker-compose up
```

Note: the GitLab server need a few minutes to start.

<!--
### 4. Update the Gitlab original Certificate

Since the initial Gitlab server **certificate** is missing some info, we may have to **regenerate** a new one and **reconfigure** in the gitlab server. Run below commands:

```dos
docker exec -it gitlab bash
mkdir /etc/gitlab/ssl_backup
mv /etc/gitlab/ssl/* /etc/gitlab/ssl_backup
cd /etc/gitlab/ssl
openssl genrsa -out ca.key 2048
openssl req -new -x509 -days 365 -key ca.key -subj "/C=CN/ST=GD/L=SZ/O=Acme, Inc./CN=Acme Root CA" -out ca.crt

# Note: Make sure to replace below `YOUR_GITLAB_DOMAIN` with our own domain name. For example, mydevopsrealprojects.com.
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

### 4. Import the gitlab new certificate in our local host CA chains

In order to make our local host be able to talk to the gitlab server via TLS, we have to import the new gitlab certificate, which is generated previous step, into our local host CA store chains. Login to our local host and run below command:

```dos
export YOUR_GITLAB_DOMAIN=mydevopsrealprojects.com
sudo docker cp gitlab:/etc/gitlab/ssl/gitlab.$YOUR_GITLAB_DOMAIN.crt /usr/local/share/ca-certificates/
sudo update-ca-certificates
```

> Note: If we are using CentOS, we may need to include "-addext basicConfstraints=critical,CA:TRUE" in the ca.crt file and use `update-ca-trust` command instead.

```dos
# For CentOS
openssl req -new -x509 -days 365 -key ca.key -addext basicConstraints=critical,CA:TRUE -subj "/C=CN/ST=GD/L=SZ/0=Acme, Inc./CN=Acme Root CA"  -out ca.crt
```
-->

### 5. Create a new project in our Gitlab server and generate a personal access token

Login to our Gitlab server website (`https://gitlab.mydevopsrealprojects.com`) and Click **"New project"** -> **"Create blank project"** -> Type a project name in **"Project name"**, e.g. *first_project*, select **"Public"** in **Visiblity Level** section -> Click **"Create project"**

![1679779492105](image/01_Y_WindowsOnly/1679779492105.png)

![1679779521707](image/01_Y_WindowsOnly/1679779521707.png)

Once the project is create, go to **"Setting""** -> **"Access Tokens"** -> Type a customized token name in **Token name** field, e.g. *terraform-token* , Select a role **"Maintainer"** in *Select a role field*, Select all scopes **"api/read_api/read_repositry/write_repository"**

![1679779621958](image/01_Y_WindowsOnly/1679779621958.png)

Make a note of the new token generated as we will need to apply it in the next step.

<!-- glpat-YtPyrWH5u1729j2HJPze -->

![1679779704707](image/01_Y_WindowsOnly/1679779704707.png)

### 6. Update `config.tfbackend`

Before running `terraform init`, we have to update `config/test/config.tfbackend` file with the credential/gitlab server info accordingly. The below is the definition for the variables:</br>

- **PROJECT_ID:** Go to the project and head to **"Setting"** -> **"General"**, and we will see **"Project ID"** in the page.
- **TF_USERNAME:** If we haven't created our own user, the default user should be `root`
- **TF_PASSWORD:** This is the gitlab **personal access token**, which we can fetch from previous step
- **TF_ADDRESS:** This is URL to store our **terraform state file**.

  The pattern is like `http://<our gitlab server url>/api/v4/projects/<our project id>/terraform/state/old-state-name`.

  For example: `http://gitlab.mydevopsrealprojects.com/api/v4/projects/${PROJECT_ID}/terraform/state/old-state-name`

![1679779555678](image/01_Y_WindowsOnly/1679779555678.png)

<!--
```dos
docker exec -it gitlab bash
cd ~/udemy-devops-real-projects/004-TerraformDockerDeployment
sudo vim config/test/config.tfbackend
```
-->

### 7. Run terraform script to deploy the infra

Init

```dos
terraform init -backend-config=config/test/config.tfbackend
```

Plan

```dos
terraform plan -var-file=config/test/test.tfvars -out deploy.tfplan
```

Apply

```dos
terraform apply deploy.tfplan
```

### 8. Verification

we should be able to visit the website via `http://gitlab.mydevopsrealprojects.com:8080`

## Clean up

Delete the terraform infra

```dos
terraform destroy -var-file=config/test/test.tfvars 
```

Delete the gitlab container, as well as the volumes mounted

```dos
docker-compose down -v
```

Delete the test container if exists

```dos
docker rm -f terraform-docker-example
```
