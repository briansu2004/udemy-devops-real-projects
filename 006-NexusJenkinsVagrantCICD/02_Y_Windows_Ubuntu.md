# Lab 006: Push War file to Nexus Repository Via Jenkins Pipeline and Deploy to Tomcat in Vagrant VM

## Lab goal

In this lab, we will set up a **Nexus** repository and push a war file from **Jenkins Pipeline**, then we will deploy the war file to a **Tomcat 9** installed in a **Vagrant VM**.

## Prerequisites

### 1. Install Docker in Windows

### 2. Install Vagrant in Windows

### 3. Create a GitHub personal access token

Classic

### 4. Get the local IP

Use `ipconfig` to find out the local IP.

e.g.

172.27.112.1

## Steps

### 1. Docker compose

```dos
cd \devbox
rd /s /q udemy-devops-real-projects
git clone https://github.com/briansu2004/udemy-devops-real-projects.git
cd udemy-devops-real-projects/006-NexusJenkinsVagrantCICD
docker-compose up
```

### 2. Fix the docker-compose issues if possible

We use the file `plugins.txt` to manage install plug-ins. However, it always forces us to use the latest versions.

Hence the 1st step `docker-compose up` will fail if there is a new version available.

e.g.

```dos
 => => naming to docker.io/library/005-vaultjenkinscicd-vault                                                                                                                               0.0s 
 => [005-vaultjenkinscicd-jenkins 2/4] COPY plugins.txt /usr/share/jenkins/ref/plugins.txt                                                                                                  0.3s 
 => ERROR [005-vaultjenkinscicd-jenkins 3/4] RUN jenkins-plugin-cli --plugin-file /usr/share/jenkins/ref/plugins.txt                                                                        5.1s 
------
 > [005-vaultjenkinscicd-jenkins 3/4] RUN jenkins-plugin-cli --plugin-file /usr/share/jenkins/ref/plugins.txt:
#0 5.036 Multiple plugin prerequisites not met:
#0 5.036 Plugin docker-workflow:528.v7c193a_0b_e67c (via pipeline-model-definition:2.2118.v31fd5b_9944b_5->git-client:4.0.0) depends on configuration-as-code:1569.vb_72405b_80249, but there is an older version defined on the t
an older version defined on the top level - configuration-as-code:1559.v38a_b_2e3b_6b_b_7,
#0 5.036 Plugin git:4.12.1 (via git-client:4.0.0) depends on configuration-as-code:1569.vb_72405b_80249, but there is an older version defined on the top level - configuration-as-code:1559.v38a_b_2e3b_6b_b_7_b_2e3b_6b_b_7
------
failed to solve: executor failed running [/bin/sh -c jenkins-plugin-cli --plugin-file /usr/share/jenkins/ref/plugins.txt]: exit code: 1
```

The solution is to update the plugin file `plugins.txt` manually based on the error messages and then re-run `docker-compose up`.

e.g.

![1672798209848](images/README/1672798209848.png)

![1672798247625](images/README/1672798247625.png)

### 3. Configure Nexus

a. Open a browser and **login to** Nexus home page (<http://127.0.0.1:8081>)

b. Fetch the **password** for `admin` user

```dos
docker exec $(docker ps --filter name=nexus-1 -q) cat /nexus-data/admin.password
```

<!--
```dos
DevOps 🚀 devbox % docker exec $(docker ps --filter name=nexus-1 -q) cat /nexus-data/admin.password
827c4aaf-5fe5-45af-a75f-540bab55a218%
```

827c4aaf-5fe5-45af-a75f-540bab55a218
-->

c. Click **"Sign In"** in the top right and type username `admin`, as well as the password fetched in the previous step

d. Follow the **wizard** and reset our password. Select **"Enable anonymous access"** and click **"Next"**->**"Finish"** to complete the guide.

<!--
DevOps2023
-->

e. Click **Gear icon** in the top and click **"Repositories"** in **"Repository"** section. Click **"Create repository"** to create a new repo.

![nexus-create-repo](images/nexus-create-repo.png)

f. Select **"maven2(hosted)"** and fill below fields as instructed:

- **Name:** `maven-nexus-repo`
- **Version policy:** Mixed
- **Deployment policy:** Allow redeploy

![nexus-create-repo-2](images/nexus-create-repo-2.png)

Click **"Create repository"** in the bottom

g. To create a new user, go to **"Security"** -> **"Users"** -> Click **"Create local user"** and fill below fields as instructed:

- **ID:** `jenkins-user`
- **First name:** Jenkins
- **Last name:** User
- **Email:** jenkins.user@gmail.com
- **Password:**  *(Type our password)*

<!--
DevOps2023
-->

- **Confirm password:**  *(Type the same password weentered above)*  
- **Status:** Active
- **Roles:** nx-admin

"Create local user"

### 4. Configure Jenkins

a. Login to our Jenkins website (<http://127.0.0.1:8080>)

<!--
b. Create First Admin User

admin

DevOps2023

admin@gmai.com
-->

b. Go to **"Manage Jenkins"** -> (Security) **"Manage Credentials"** -> **"System"** -> **"Global credentials (unrestricted)"** -> Click **"Add Credentials"** and fill out the page in below selection:

<!--
> Note: The **username** and **password** is in `.env` file

admin
DevOps2023

or

admin
bitnami
-->

- **Kind:** Username with password
- **Scope:** Global(Jenkins, nodes, items, all child items, etc)
- **Username:** jenkins-user
- **Password:** *(Type the password weset in previous step)*
- **ID:** nexus
- **Description:** nexus credential

![Jenkins Credentials](images/jenkins-01.png)

c. Add github credential in Jenkins

Go to **"Manage Jenkins"** -> **"Manage Credentials"** ->  **"System"** -> **"Global credentials (unrestricted)"** -> Click **"Add Credentials"** and fill out the page below below selection:

- **Scope:** Global (Jenkins,nodes,items,all child items,etc)
- **Username:** (the github username)
- **Password:** `<the github personal access token>`
- **ID:** (the id which will be referred in Jenkinsfile, e.g. github-token)
- **Description:** Github token

<!--
briansu2004
github-token
-->

<!--
github
profile photo
settings
Developer settings
Personal access tokens
Tokens (classic)
Generate new token
Generate new token (classic)
https://github.com/settings/tokens/new
Note
Expiration
Scopes
Generate token
Copy
-->

![Add github credential in Jenkins](images/README/github-credentials.png)

![Global credentials (unrestricted)](images/README/global-credentials.png)

d. To create a new pipeline, go back to Dashboard, click **"New Item"** in the left navigation lane, and type the item name (i.g. `first-project`) and select **"Pipeline"**. Click **"OK"** to configure the pipeline.

e. Go to **"Pipeline"** section and select **"Pipeline script from SCM"** in the **"Definition"** field

f. Select **"Git"** in **"SCM"** field

g. Add `https://github.com/briansu2004/udemy-devops-real-projects.git` in **"Repository URL"** field

h. Select our github credential in **"Credentials"**

i. Type `*/main` in **"Branch Specifier"** field

j. Type `006-NexusJenkinsVagrantCICD/Jenkinsfile` in **"Script Path"**

k. Unselect **"Lightweight checkout"** and click "Apply" to complete the creation

l. To add maven tool, go back to **"Dashboard"** -> **"Manage Jenkins"** -> **"Tool"** -> Scroll down to **"Maven"** section and click **"Add Maven"**. Then fill out below fields as instructed:

- **Name:** m3
- **Install automaticall** selected
- **Version:** 3.8.6

Click **"Save"**

### 5. Launch the Jenkins pipeline

Go to **"Dashboard"** -> Click **"first-project"** pipeline -> Click **"Build Now"**, then the Jenkins pipeline will compile the app to a war file and upload to the Nexus repository.

### 6. Check the war file in the Nexus repo

Login to the Nexus website (<http://127.0.0.1:8081>) and go to **"Browse"** section, and then click **"maven-nexus-repo"**, we should be able to see the artifacts just uploaded.

![nexus-repo-configuration](images/nexus-repo-configuration.png)

Click the war file and copy the link.

![nexus-war-download-url](images/nexus-war-download-url.png)

<!--
http://127.0.0.1:8081/repository/maven-nexus-repo/sparkjava-hello-world/sparkjava-hello-world/1.0/sparkjava-hello-world-1.0.war
-->

Replace the IP address `127.0.0.1` to `10.0.0.124`.

<!--
the actual IP of our host (running `ifconfig` to check our host IP).
-->

e.g.

<http://127.0.0.1:8081/repository/maven-nexus-repo/sparkjava-hello-world/sparkjava-hello-world/1.0/sparkjava-hello-world-1.0.war>

->

<http://172.27.112.1:8081/repository/maven-nexus-repo/sparkjava-hello-world/sparkjava-hello-world/1.0/sparkjava-hello-world-1.0.war>

### 7. Deploy a Tomcat server via Vagrant

Start the Vagrant VM

```dos
vagrant up
```

Login to the Tomcat Vagrant VM

```dos
vagrant ssh
```

## 8. Verify the sparkjava-hello-world app is down :-)

This URL should not work and it is expected.

```dos
http://127.0.0.1:8088/sparkjava-hello-world-1.0/hello
```

## 9. Download the war file and deploy to the Tomcat server

Download the war from the Nexus repo.

```dos
cd /var/lib/tomcat9/webapps/
sudo wget http://10.0.0.124:8081/repository/maven-nexus-repo/sparkjava-hello-world/sparkjava-hello-world/1.0/sparkjava-hello-world-1.0.war 
```

<!--
```dos
vagrant@vagrant:/var/lib/tomcat9/webapps$ sudo wget http://10.0.0.124:8081/repository/maven-nexus-repo/sparkjava-hello-world/sparkjava-hello-world/1.0/sparkjava-hello-world-1.0.war 
--2023-04-01 21:32:59--  http://10.0.0.124:8081/repository/maven-nexus-repo/sparkjava-hello-world/sparkjava-hello-world/1.0/sparkjava-hello-world-1.0.war
Connecting to 10.0.0.124:8081... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2427996 (2.3M) [application/java-archive]
Saving to: ‘sparkjava-hello-world-1.0.war’

sparkjava-hello-world-1.0.war    100%[==========================================================>]   2.32M  --.-KB/s    in 0.05s   

2023-04-01 21:32:59 (51.4 MB/s) - ‘sparkjava-hello-world-1.0.war’ saved [2427996/2427996]
```
--->

The war file will be unzipped (note: we may have to wait for 2 mins).

```dos
vagrant@vagrant:/var/lib/tomcat9/webapps$ ls
ROOT  sparkjava-hello-world-1.0  sparkjava-hello-world-1.0.war
```

## 10. Verify the sparkjava-hello-world app is down :-)

Type `exit` to exit the Vagrant VM.

Type below URL in yoru browser, and we should be able to see the "Hello World" page.

```dos
http://127.0.0.1:8088/sparkjava-hello-world-1.0/hello
```

![helloworld](images/helloworld.png)
