# Project 004: Deploy Docker with Terraform Script

Windows + Ubuntu

## Prerequisites

### 1. Install and start Vagrant

```dos
vagrant up 
vagrant ssh
```

### 2. Install Docker and Docker Compose in Vagrant

```dos
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

```dos
...
```

## Steps

## 1. Config the GitLab domain name

In this lab, we will use `mydevopsrealprojects.com` as the GitLab domain name.

Hence the GitLab server URL is `http://gitlab.mydevopsrealprojects.com`

## 2. Configure the **hosts** file

- Add these 2 entries in Windows's hosts file `C:\Windows\System32\drivers\etc\hosts`

```dos
192.168.33.10 gitlab.mydevopsrealprojects.com
192.168.33.10 registry.gitlab.mydevopsrealprojects.com
```

- Add these 2 entries in Vagrant Ubuntu's hosts file `/etc/hosts`

```bash
0.0.0.0 gitlab.mydevopsrealprojects.com
0.0.0.0 registry.gitlab.mydevopsrealprojects.com
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

## 3. [Ubuntu] Start the docker compose

```bash
cd
rm -rf udemy-devops-real-projects
git clone https://github.com/briansu2004/udemy-devops-real-projects.git
cd udemy-devops-real-projects/004-TerraformDockerDeployment
docker-compose up
```

Note: the GitLab server need a few minutes to start.

<!--
### 1. Deploy a gitlab server to store the terraform state file

```dos
git clone https://github.com/briansu2004/udemy-devops-real-projects.git
cd udemy-devops-real-projects/004-TerraformDockerDeployment
docker-compose up
```

> Note: Once the gitlab container is fully up running, we can run below command to retrieve the initial password, if we haven't specified it in the deployment file. The default admin username should be `root`

```dos
sudo docker exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password
```

### 2. Add the new DNS record in our local hosts file

In our `docker-compose.yaml`, we have defined our gitlab server hostname in `hostname` field. Add it to our local hosts file so that we can use it to git clone the repo from our gitlab server.

```dos
export Your_Local_Host_IP=<Your_Local_Host_IP>
echo "${Your_Local_Host_IP}  gitlab.mydevopsrealprojects.com" | sudo tee -a /etc/hosts
```

e.g.

```dos
echo "${Your_Local_Host_IP}  gitlab.mydevopsrealprojects.com" | sudo tee -a /etc/hosts
```

Then we should be able to access the Gitlab website via `https://mydevopsrealprojects.com`
-->

### 4. Update the Gitlab original Certificate

Since the initial Gitlab server **certificate** is missing some info, we may have to **regenerate** a new one and **reconfigure** in the gitlab server. Run below commands:

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
```bash
root@gitlab:/etc/gitlab/ssl# ls -l
total 36
-rw-r--r-- 1 root root 1285 Mar 25 23:29 ca.crt
-rw------- 1 root root 1675 Mar 25 23:29 ca.key
-rw-r--r-- 1 root root   41 Mar 25 23:30 ca.srl
-rw-r--r-- 1 root root 1289 Mar 25 23:29 gitlab.mydevopsrealprojects.com.crt
-rw-r--r-- 1 root root  997 Mar 25 23:29 gitlab.mydevopsrealprojects.com.csr
-rw------- 1 root root 1704 Mar 25 23:29 gitlab.mydevopsrealprojects.com.key
-rw-r--r-- 1 root root 1346 Mar 25 23:30 registry.gitlab.mydevopsrealprojects.com.crt
-rw-r--r-- 1 root root  997 Mar 25 23:30 registry.gitlab.mydevopsrealprojects.com.csr
-rw------- 1 root root 1704 Mar 25 23:30 registry.gitlab.mydevopsrealprojects.com.key
```
-->

Note: the GitLab server need a few minutes to start.

<!--
Need to restart docker container???

docker restart gitlab
docker compose up

vagrant@vagrant:~/udemy-devops-real-projects/004-TerraformDockerDeployment$ docker ps
CONTAINER ID   IMAGE                     COMMAND             CREATED          STATUS                    PORTS
           NAMES
126b1543f62b   gitlab/gitlab-ce:latest   "/assets/wrapper"   46 minutes ago   Up 46 minutes (healthy)   0.0.0.0:80->80/tcp, :::80->80/tcp, 0.0.0.0:443->443/tcp, :::443->443/tcp, 0.0.0.0:2222->22/tcp, :::2222->22/tcp   gitlab
vagrant@vagrant:~/udemy-devops-real-projects/004-TerraformDockerDeployment$ docker restart
"docker restart" requires at least 1 argument.
See 'docker restart --help'.

Usage:  docker restart [OPTIONS] CONTAINER [CONTAINER...]

Restart one or more containers
vagrant@vagrant:~/udemy-devops-real-projects/004-TerraformDockerDeployment$ docker restart 12
12
-->

### 5. Import the gitlab new certificate in our local host CA chains

In order to make our local host be able to talk to the gitlab server via TLS, we have to import the new gitlab certificate, which is generated previous step, into our local host CA store chains. Login to our local host and run below command:

```dos
export YOUR_GITLAB_DOMAIN=mydevopsrealprojects.com
sudo docker cp gitlab:/etc/gitlab/ssl/gitlab.$YOUR_GITLAB_DOMAIN.crt /usr/local/share/ca-certificates/
sudo update-ca-certificates
```

<!--
```bash
vagrant@vagrant:~/udemy-devops-real-projects/004-TerraformDockerDeployment$ export YOUR_GITLAB_DOMAIN=mydevopsrealprojects.com
vagrant@vagrant:~/udemy-devops-real-projects/004-TerraformDockerDeployment$ sudo docker cp gitlab:/etc/gitlab/ssl/gitlab.$YOUR_GITLAB_DOMAIN.crt /usr/local/share/ca-certificates/
Preparing to copy...
Successfully copied 3.072kB to /usr/local/share/ca-certificates/
vagrant@vagrant:~/udemy-devops-real-projects/004-TerraformDockerDeployment$
vagrant@vagrant:~/udemy-devops-real-projects/004-TerraformDockerDeployment$ sudo update-ca-certificates
Updating certificates in /etc/ssl/certs...
rehash: warning: skipping ca-certificates.crt,it does not contain exactly one certificate or CRL
1 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.
```
-->

### 6. Create a new project in our Gitlab server and generate a personal access token

Login to our Gitlab server website (`https://gitlab.mydevopsrealprojects.com`) and Click **"New project"** -> **"Create blank project"** -> Type a project name in **"Project name"**, e.g. *first_project*, select **"Public"** in **Visiblity Level** section -> Click **"Create project"**

![1679781263992](image/02_Y_Windows_Ubuntu/1679781263992.png)

![1679779492105](image/01_Y_WindowsOnly/1679779492105.png)

![1679779521707](image/01_Y_WindowsOnly/1679779521707.png)

Once the project is create, go to **"Setting""** -> **"Access Tokens"** -> Type a customized token name in **Token name** field, e.g. *terraform-token* , Select a role **"Maintainer"** in *Select a role field*, Select all scopes **"api/read_api/read_repositry/write_repository"**

![1679779621958](image/01_Y_WindowsOnly/1679779621958.png)

Make a note of the new token generated as we will need to apply it in the next step.

![1679779704707](image/01_Y_WindowsOnly/1679779704707.png)

<!-- glpat-moigx-sCKzk2KzG9YGP_ -->

GitLab API verification with token:

```bash
curl --header "Private-Token: glpat-UnvMJckVQ_xFsny7rhCC" https://gitlab.mydevopsrealprojects.com/api/v4/projects

curl --header "Private-Token: glpat-moigx-sCKzk2KzG9YGP_" http://gitlab.mydevopsrealprojects.com/api/v4/projects
```

### 7. Update `config.tfbackend`

Before running `terraform init`, we have to update `config/test/config.tfbackend` file with the credential/gitlab server info accordingly. The below is the definition for the variables:</br>

- **PROJECT_ID:** Go to the project and head to **"Setting"** -> **"General"**, and we will see **"Project ID"** in the page.
- **TF_USERNAME:** If we haven't created our own user, the default user should be `root`
- **TF_PASSWORD:** This is the gitlab **personal access token**, which we can fetch from previous step
- **TF_ADDRESS:** This is URL to store our **terraform state file**.

  The pattern is like `https://<our gitlab server url>/api/v4/projects/<our project id>/terraform/state/old-state-name`.

  For example: `https://gitlab.mydevopsrealprojects.com/api/v4/projects/${PROJECT_ID}/terraform/state/old-state-name`

![1679779555678](image/01_Y_WindowsOnly/1679779555678.png)

### 8. Run terraform script to deploy the infra

<!--
This is a quick and dirty way to clean up Terraform state -

```bash
rm -rf .terraform
```
-->

Init

```dos
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

```dos
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
```
-->

Apply

```dos
terraform apply deploy.tfplan
```

<!--
```bash
vagrant@vagrant:~/udemy-devops-real-projects/004-TerraformDockerDeployment$ terraform apply deploy.tfplan
docker_image.hello_world: Creating...
docker_image.hello_world: Still creating... [10s elapsed]
docker_image.hello_world: Still creating... [20s elapsed]
docker_image.hello_world: Creation complete after 21s [id=sha256:3681d33a9e299fe6788a9c03c5b8f365c59638899539ef5021c3b587e92fe967hello-world]
docker_container.hello_world: Creating...
docker_container.hello_world: Creation complete after 1s [id=199bfef72df9e1e521cb80a20d81ecf375fd7abbd6c476f89382e9471c08aed6]
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

### 9. Verification

we should be able to visit the website via `http://gitlab.mydevopsrealprojects.com:8080`

![Hello World!](image/02_Y_Windows_Ubuntu/1679869430545.png)

## Clean up

Delete the terraform infra

```dos
terraform destroy -var-file=config/test/test.tfvars 
```

Delete the gitlab container, as well as the volumes mounted

```dos
docker-compose down -v
```