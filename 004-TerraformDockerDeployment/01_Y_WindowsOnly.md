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

<!-- glpat-n11zSaXKeKSXdVubLgyv -->

```dos
curl --header "Private-Token: glpat-n11zSaXKeKSXdVubLgyv" http://gitlab.mydevopsrealprojects.com/api/v4/projects
```

![1679779704707](image/01_Y_WindowsOnly/1679779704707.png)

<!--
```dos

C:\devbox>curl --header "Private-Token: glpat-n11zSaXKeKSXdVubLgyv" http://gitlab.mydevopsrealprojects.com/api/v4/projects
[{"id":2,"description":null,"name":"first_project","name_with_namespace":"GitLab Instance / first_project","path":"first_project","path_with_namespace":"gitlab-instance-c345c4f2/first_project","created_at":"2023-03-26T21:33:20.376Z","default_branch":"main","tag_list":[],"topics":[],"ssh_url_to_repo":"git@gitlab.mydevopsrealprojects.com:gitlab-instance-c345c4f2/first_project.git","http_url_to_repo":"http://gitlab.mydevopsrealprojects.com/gitlab-instance-c345c4f2/first_project.git","web_url":"http://gitlab.mydevopsrealprojects.com/gitlab-instance-c345c4f2/first_project","readme_url":"http://gitlab.mydevopsrealprojects.com/gitlab-instance-c345c4f2/first_project/-/blob/main/README.md","avatar_url":null,"forks_count":0,"star_count":0,"last_activity_at":"2023-03-26T21:33:20.376Z","namespace":{"id":2,"name":"GitLab Instance","path":"gitlab-instance-c345c4f2","kind":"group","full_path":"gitlab-instance-c345c4f2","parent_id":null,"avatar_url":null,"web_url":"http://gitlab.mydevopsrealprojects.com/groups/gitlab-instance-c345c4f2"},"_links":{"self":"http://gitlab.mydevopsrealprojects.com/api/v4/projects/2","issues":"http://gitlab.mydevopsrealprojects.com/api/v4/projects/2/issues","merge_requests":"http://gitlab.mydevopsrealprojects.com/api/v4/projects/2/merge_requests","repo_branches":"http://gitlab.mydevopsrealprojects.com/api/v4/projects/2/repository/branches","labels":"http://gitlab.mydevopsrealprojects.com/api/v4/projects/2/labels","events":"http://gitlab.mydevopsrealprojects.com/api/v4/projects/2/events","members":"http://gitlab.mydevopsrealprojects.com/api/v4/projects/2/members","cluster_agents":"http://gitlab.mydevopsrealprojects.com/api/v4/projects/2/cluster_agents"},"packages_enabled":true,"empty_repo":false,"archived":false,"visibility":"internal","resolve_outdated_diff_discussions":false,"container_expiration_policy":{"cadence":"1d","enabled":false,"keep_n":10,"older_than":"90d","name_regex":".*","name_regex_keep":null,"next_run_at":"2023-03-27T21:33:20.434Z"},"issues_enabled":true,"merge_requests_enabled":true,"wiki_enabled":true,"jobs_enabled":true,"snippets_enabled":true,"container_registry_enabled":true,"service_desk_enabled":false,"service_desk_address":null,"can_create_merge_request_in":true,"issues_access_level":"enabled","repository_access_level":"enabled","merge_requests_access_level":"enabled","forking_access_level":"enabled","wiki_access_level":"enabled","builds_access_level":"enabled","snippets_access_level":"enabled","pages_access_level":"enabled","operations_access_level":"enabled","analytics_access_level":"enabled","container_registry_access_level":"enabled","security_and_compliance_access_level":"private","emails_disabled":null,"shared_runners_enabled":true,"lfs_enabled":true,"creator_id":1,"import_url":null,"import_type":null,"import_status":"none","open_issues_count":0,"ci_default_git_depth":20,"ci_forward_deployment_enabled":true,"ci_job_token_scope_enabled":false,"ci_separated_caches":true,"ci_opt_in_jwt":false,"ci_allow_fork_pipelines_to_run_in_parent_project":true,"public_jobs":true,"build_timeout":3600,"auto_cancel_pending_pipelines":"enabled","ci_config_path":null,"shared_with_groups":[],"only_allow_merge_if_pipeline_succeeds":false,"allow_merge_on_skipped_pipeline":null,"restrict_user_defined_variables":false,"request_access_enabled":true,"only_allow_merge_if_all_discussions_are_resolved":false,"remove_source_branch_after_merge":true,"printing_merge_request_link_enabled":true,"merge_method":"merge","squash_option":"default_off","enforce_auth_checks_on_uploads":true,"suggestion_commit_message":null,"merge_commit_template":null,"squash_commit_template":null,"auto_devops_enabled":true,"auto_devops_deploy_strategy":"continuous","autoclose_referenced_issues":true,"keep_latest_artifact":true,"runner_token_expiration_interval":null,"permissions":{"project_access":{"access_level":40,"notification_level":3},"group_access":null}},{"id":1,"description":"This project is automatically generated and helps monitor this GitLab instance. [Learn more](/help/administration/monitoring/gitlab_self_monitoring_project/index).","name":"Monitoring","name_with_namespace":"GitLab Instance / Monitoring","path":"Monitoring","path_with_namespace":"gitlab-instance-c345c4f2/Monitoring","created_at":"2023-03-26T21:30:35.415Z","default_branch":"main","tag_list":[],"topics":[],"ssh_url_to_repo":"git@gitlab.mydevopsrealprojects.com:gitlab-instance-c345c4f2/Monitoring.git","http_url_to_repo":"http://gitlab.mydevopsrealprojects.com/gitlab-instance-c345c4f2/Monitoring.git","web_url":"http://gitlab.mydevopsrealprojects.com/gitlab-instance-c345c4f2/Monitoring","readme_url":null,"avatar_url":null,"forks_count":0,"star_count":0,"last_activity_at":"2023-03-26T21:30:35.415Z","namespace":{"id":2,"name":"GitLab Instance","path":"gitlab-instance-c345c4f2","kind":"group","full_path":"gitlab-instance-c345c4f2","parent_id":null,"avatar_url":null,"web_url":"http://gitlab.mydevopsrealprojects.com/groups/gitlab-instance-c345c4f2"},"_links":{"self":"http://gitlab.mydevopsrealprojects.com/api/v4/projects/1","issues":"http://gitlab.mydevopsrealprojects.com/api/v4/projects/1/issues","merge_requests":"http://gitlab.mydevopsrealprojects.com/api/v4/projects/1/merge_requests","repo_branches":"http://gitlab.mydevopsrealprojects.com/api/v4/projects/1/repository/branches","labels":"http://gitlab.mydevopsrealprojects.com/api/v4/projects/1/labels","events":"http://gitlab.mydevopsrealprojects.com/api/v4/projects/1/events","members":"http://gitlab.mydevopsrealprojects.com/api/v4/projects/1/members","cluster_agents":"http://gitlab.mydevopsrealprojects.com/api/v4/projects/1/cluster_agents"},"packages_enabled":true,"empty_repo":true,"archived":false,"visibility":"internal","resolve_outdated_diff_discussions":false,"container_expiration_policy":{"cadence":"1d","enabled":false,"keep_n":10,"older_than":"90d","name_regex":".*","name_regex_keep":null,"next_run_at":"2023-03-27T21:30:35.596Z"},"issues_enabled":true,"merge_requests_enabled":true,"wiki_enabled":true,"jobs_enabled":true,"snippets_enabled":true,"container_registry_enabled":true,"service_desk_enabled":false,"can_create_merge_request_in":true,"issues_access_level":"enabled","repository_access_level":"enabled","merge_requests_access_level":"enabled","forking_access_level":"enabled","wiki_access_level":"enabled","builds_access_level":"enabled","snippets_access_level":"enabled","pages_access_level":"private","operations_access_level":"enabled","analytics_access_level":"enabled","container_registry_access_level":"enabled","security_and_compliance_access_level":"private","emails_disabled":null,"shared_runners_enabled":true,"lfs_enabled":true,"creator_id":1,"import_status":"none","open_issues_count":0,"ci_default_git_depth":20,"ci_forward_deployment_enabled":true,"ci_job_token_scope_enabled":false,"ci_separated_caches":true,"ci_opt_in_jwt":false,"ci_allow_fork_pipelines_to_run_in_parent_project":true,"public_jobs":true,"build_timeout":3600,"auto_cancel_pending_pipelines":"enabled","ci_config_path":null,"shared_with_groups":[],"only_allow_merge_if_pipeline_succeeds":false,"allow_merge_on_skipped_pipeline":null,"restrict_user_defined_variables":false,"request_access_enabled":true,"only_allow_merge_if_all_discussions_are_resolved":false,"remove_source_branch_after_merge":true,"printing_merge_request_link_enabled":true,"merge_method":"merge","squash_option":"default_off","enforce_auth_checks_on_uploads":true,"suggestion_commit_message":null,"merge_commit_template":null,"squash_commit_template":null,"auto_devops_enabled":true,"auto_devops_deploy_strategy":"continuous","autoclose_referenced_issues":true,"keep_latest_artifact":true,"runner_token_expiration_interval":null,"permissions":{"project_access":null,"group_access":null}}]
C:\devbox>
```
-->

### 6. Update `config.tfbackend`

Before running `terraform init`, we have to update `config/test/config.tfbackend` file with the credential/gitlab server info accordingly. The below is the definition for the variables:</br>

- **PROJECT_ID:** Go to the project and head to **"Setting"** -> **"General"**, and we will see **"Project ID"** in the page.
- **TF_USERNAME:** If we haven't created our own user, the default user should be `root`
- **TF_PASSWORD:** This is the gitlab **personal access token**, which we can fetch from previous step
- **TF_ADDRESS:** This is URL to store our **terraform state file**.

![1679779555678](image/01_Y_WindowsOnly/1679779555678.png)

### 7. Run terraform script to deploy the infra

Init

```dos
git clone https://github.com/briansu2004/udemy-devops-real-projects.git
cd udemy-devops-real-projects/004-TerraformDockerDeployment
type config/test/config.tfbackend

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
