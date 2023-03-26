# Project 004: Deploy Docker with Terraform Script

Mac + Ubuntu

<!--
Issues:

There are issues to install Terraform in Ubuntu (in Mac).

But no issues to nstall Terraform in Ubuntu (in Windows)!

This issue happens from time to time and it is very annoying!

Wasted my whole day yesterday but it works today!
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

### 3. Install Terraform in Vagrant

...

## Steps

### 1. Config the GitLab domain name

In this lab, we will use `mydevopsrealprojects.com` as the GitLab domain name.

Hence the gitlab URL is `http://gitlab.mydevopsrealprojects.com`

We need it in the `docker-compose.yml` file.

### 2. Configure the **hosts** file

Add these 2 entries in Vagrant Ubuntu's hosts file `/etc/hosts` -

```bash
0.0.0.0 gitlab.mydevopsrealprojects.com
0.0.0.0 registry.gitlab.mydevopsrealprojects.com
```

<!--
```bash
127.0.0.1 gitlab.mydevopsrealprojects.com
127.0.0.1 registry.gitlab.mydevopsrealprojects.com
```
-->

Add these 2 entries in Mac's hosts file `/etc/hosts` -

```bash
192.168.33.10 gitlab.mydevopsrealprojects.com
192.168.33.10 registry.gitlab.mydevopsrealprojects.com
```

### 3. Docker compose

```bash
git clone https://github.com/briansu2004/udemy-devops-real-projects.git
cd udemy-devops-real-projects/004-TerraformDockerDeployment
docker-compose up
```

### 4. Update the Gitlab original Certificate

Since the initial Gitlab server **certificate** is missing some info, you may have to **regenerate** a new one and **reconfigure** in the gitlab server. Run below commands:

```bash
docker exec -it gitlab bash

mkdir /etc/gitlab/ssl
cd /etc/gitlab/ssl

# ca.key
openssl genrsa -out ca.key 2048

# ca.crt
openssl req -new -x509 -days 365 -key ca.key -subj "/C=CN/ST=GD/L=SZ/O=Acme, Inc./CN=Acme Root CA" -out ca.crt

export YOUR_GITLAB_DOMAIN=mydevopsrealprojects.com

# Certificate for gitlab server
# gitlab.mydevopsrealprojects.com.csr + gitlab.mydevopsrealprojects.com.key
openssl req -newkey rsa:2048 -nodes -keyout gitlab.$YOUR_GITLAB_DOMAIN.key -subj "/C=CN/ST=GD/L=SZ/O=Acme, Inc./CN=*.$YOUR_GITLAB_DOMAIN" -out gitlab.$YOUR_GITLAB_DOMAIN.csr

# ca.srl + gitlab.mydevopsrealprojects.com.crt
openssl x509 -req -extfile <(printf "subjectAltName=DNS:$YOUR_GITLAB_DOMAIN,DNS:gitlab.$YOUR_GITLAB_DOMAIN") -days 365 -in gitlab.$YOUR_GITLAB_DOMAIN.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out gitlab.$YOUR_GITLAB_DOMAIN.crt

# Certificate for container registry
# registry.gitlab.mydevopsrealprojects.com.csr + registry.gitlab.mydevopsrealprojects.com.key
openssl req -newkey rsa:2048 -nodes -keyout registry.gitlab.$YOUR_GITLAB_DOMAIN.key -subj "/C=CN/ST=GD/L=SZ/O=Acme, Inc./CN=*.$YOUR_GITLAB_DOMAIN" -out registry.gitlab.$YOUR_GITLAB_DOMAIN.csr

# registry.gitlab.mydevopsrealprojects.com.crt
  openssl x509 -req -extfile <(printf "subjectAltName=DNS:$YOUR_GITLAB_DOMAIN,DNS:gitlab.$YOUR_GITLAB_DOMAIN,DNS:registry.gitlab.$YOUR_GITLAB_DOMAIN") -days 365 -in registry.gitlab.$YOUR_GITLAB_DOMAIN.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out registry.gitlab.$YOUR_GITLAB_DOMAIN.crt

gitlab-ctl reconfigure

gitlab-ctl restart

exit
```

<!--
### 5. Restart GitLab

```bash
docker compose down
docker compose up
```
-->

### 6. Import the gitlab new certificate in your local host CA chains

In order to make your local host be able to talk to the gitlab server via TLS, you have to import the new gitlab certificate, which is generated previous step, into your local host CA store chains. Login to your local host and run below command:

```bash
export YOUR_GITLAB_DOMAIN=mydevopsrealprojects.com

sudo docker cp gitlab:/etc/gitlab/ssl/gitlab.$YOUR_GITLAB_DOMAIN.crt /usr/local/share/ca-certificates/

sudo docker cp gitlab:/etc/gitlab/ssl/registry.gitlab.mydevopsrealprojects.com.crt /usr/local/share/ca-certificates/

sudo docker cp gitlab:/etc/gitlab/ssl/ca.crt /usr/local/share/ca-certificates/

ls -l /usr/local/share/ca-certificates/

sudo update-ca-certificates
```

<!--
vagrant@vagrant:~$ sudo docker cp gitlab:/etc/gitlab/ssl/gitlab.$YOUR_GITLAB_DOMAIN.crt /usr/local/share/ca-certificates/
Preparing to copy...
Successfully copied 3.072kB to /usr/local/share/ca-certificates/
vagrant@vagrant:~$ 
vagrant@vagrant:~$ ls -l !$
ls -l /usr/local/share/ca-certificates/
total 4
-rw-r--r-- 1 root root 1289 Mar 26 19:17 gitlab.mydevopsrealprojects.com.crt
vagrant@vagrant:~$ 
vagrant@vagrant:~$ sudo update-ca-certificates
Updating certificates in /etc/ssl/certs...
rehash: warning: skipping ca-certificates.crt,it does not contain exactly one certificate or CRL
1 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.
-->

### 7. Create a new project in your Gitlab server and generate a personal access token

Login to your Gitlab server website (`https://mydevopsrealprojects.com`) and Click **"New project"** -> **"Create blank project"** -> Type a project name in **"Project name"**, i.g. *first_project*, select **"Public"** in **Visiblity Level** section -> Click **"Create project"** </br>

Once the project is create, go to **"Setting""** -> **"Access Tokens"** -> Type a customized token name in **Token name** field, i.ig  *terraform-token* , Select a role **"Maintainer"** in *Select a role field*, Select scopes **"api/read_api/read_repositry/write_repository"** </br>

Make a note of the new token generated as you will need to apply it in the next step.

<!-- glpat-UnvMJckVQ_xFsny7rhCC-->

GitLab API verification with token:

```bash
curl --header "Private-Token: glpat-UnvMJckVQ_xFsny7rhCC" https://gitlab.mydevopsrealprojects.com/api/v4/projects


curl --header "Private-Token: glpat-fpMjQW8LB8ZKjskLFCW1" https://gitlab.mydevopsrealprojects.com/api/v4/projects
```

![gitlab-personal-accees-token](images/gitlab-personal-accees-token.png)

### 8. Update `config.tfbackend`

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

### 9. Run terraform script to deploy the infra

Init

```bash
git clone https://github.com/briansu2004/udemy-devops-real-projects.git
cd udemy-devops-real-projects/004-TerraformDockerDeployment
cat config/test/config.tfbackend
rm -rf .terraform
terraform init -backend-config=config/test/config.tfbackend
```

<!--
```bash
vagrant@vagrant:~/udemy-devops-real-projects/004-TerraformDockerDeployment$ terraform init -backend-config=config/test/config.tfbackend

Initializing the backend...

Successfully configured the backend "http"! Terraform will automatically
use this backend unless the backend configuration changes.

Initializing provider plugins...
- Finding kreuzwerker/docker versions matching "~> 2.13.0"...
- Installing kreuzwerker/docker v2.13.0...
- Installed kreuzwerker/docker v2.13.0 (self-signed, key ID 24E54F214569A8A5)

Partner and community providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://www.terraform.io/docs/cli/plugins/signing.html

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```
-->

Plan

```bash
terraform plan -var-file=config/test/test.tfvars -out deploy.tfplan
```

<!--
```bash
vagrant@vagrant:~/udemy-devops-real-projects/004-TerraformDockerDeployment$ terraform plan -var-file=config/test/test.tfvars -out deploy.tfplan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # docker_container.hello_world will be created
  + resource "docker_container" "hello_world" {
      + attach           = false
      + bridge           = (known after apply)
      + command          = (known after apply)
      + container_logs   = (known after apply)
      + entrypoint       = (known after apply)
      + env              = (known after apply)
      + exit_code        = (known after apply)
      + gateway          = (known after apply)
      + hostname         = (known after apply)
      + id               = (known after apply)
      + image            = (known after apply)
      + init             = (known after apply)
      + ip_address       = (known after apply)
      + ip_prefix_length = (known after apply)
      + ipc_mode         = (known after apply)
      + log_driver       = "json-file"
      + logs             = false
      + must_run         = true
      + name             = "terraform-docker-example"
      + network_data     = (known after apply)
      + read_only        = false
      + remove_volumes   = true
      + restart          = "no"
      + rm               = false
      + security_opts    = (known after apply)
      + shm_size         = (known after apply)
      + start            = true
      + stdin_open       = false
      + tty              = false

      + ports {
          + external = 8080
          + internal = 8080
          + ip       = "0.0.0.0"
          + protocol = "tcp"
        }
    }

  # docker_image.hello_world will be created
  + resource "docker_image" "hello_world" {
      + id          = (known after apply)
      + latest      = (known after apply)
      + name        = "hello-world"
      + output      = (known after apply)
      + repo_digest = (known after apply)

      + build {
          + dockerfile = "Dockerfile"
          + label      = {
              + "author" = "devopsdaydayup"
            }
          + path       = "."
          + remove     = true
          + tag        = [
              + "hello-world:develop",
            ]
        }
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + docker_container_name = "terraform-docker-example"
╷
│ Warning: Deprecated attribute
│ 
│   on containers.tf line 2, in resource "docker_container" "hello_world":
│    2:   image = docker_image.hello_world.latest
│ 
│ The attribute "latest" is deprecated. Refer to the provider documentation for details.
│ 
│ (and one more similar warning elsewhere)
╵

────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Saved the plan to: deploy.tfplan

To perform exactly these actions, run the following command to apply:
    terraform apply "deploy.tfplan"
```
-->

Apply

```bash
terraform apply deploy.tfplan
```

<!--
```bash
vagrant@vagrant:~/udemy-devops-real-projects/004-TerraformDockerDeployment$ terraform apply deploy.tfplan
docker_image.hello_world: Creating...
docker_image.hello_world: Still creating... [10s elapsed]
docker_image.hello_world: Creation complete after 18s [id=sha256:7720ead7de527ed387e8a3e8c0b8fcfb60e154eba46aa0a6b958811c82b4b8f4hello-world]
docker_container.hello_world: Creating...
docker_container.hello_world: Creation complete after 1s [id=736e580d74cc611185f5297139118d822af7085b5d8ba4d02c2dc1a61fa6c105]
╷
│ Warning: Deprecated attribute
│
│   on containers.tf line 2, in resource "docker_container" "hello_world":
│    2:   image = docker_image.hello_world.latest
│
│ The attribute "latest" is deprecated. Refer to the provider documentation for details.
│
│ (and one more similar warning elsewhere)
╵

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

Outputs:

docker_container_name = "terraform-docker-example"
```
-->

### 10. Import certificate to local Mac

...

### 11. Verification

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
