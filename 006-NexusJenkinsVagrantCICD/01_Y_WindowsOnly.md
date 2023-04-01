# Project 006: Push War file to Nexus Repository Via Jenkins Pipeline and Deploy to Tomcat in Vagrant VM

## Project Goal

In this lab, we will set up a **Nexus** repository and push a war file from **Jenkins Pipeline**, then we will deploy the war file to a **Tomcat 9** installed in a **Vagrant VM**.

## Steps

### 1. Deploy Jenins and Nexus containers

```dos
docker-compose build
docker-compose up
```

### 2. Configure Nexus

a. Open a browser and **login to** Nexus home page (<http://0.0.0.0:8081>)

b. Fetch the **password** for `admin` user

```dos
docker exec $(docker ps --filter name=nexus_1 -q) cat /nexus-data/admin.password
```

c. Click **"Sign In"** in the top right and type username `admin`, as well as the password fetched in the previous step

d. Follow the **wizard** and reset our password. Select **"Enable anonymous access"** and click **"Next"**->**"Finish"** to complete the guide.

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
- **Confirm password:**  *(Type the same password weentered above)*  
- **Status:** Active
- **Roles:** nx-admin

### 3. Configure Jenkins

a. Login to our Jenkins website (<http://0.0.0.0:8080>) and go to **"Manage Jenkins"** -> **"Manage Credentials"** ->  **"System"** -> **"Global credentials (unrestricted)"** -> Click **"Add Credentials"** and weshould fill out the page in below selection:

> Note: The **username** and **password** is in `.env` file

- **Kind:** Username with password
- **Scope:** Global(Jenkins, nodes, items, all child items, etc)
- **Username:** jenkins-user
- **Password:** *(Type the password weset in previous step)*
- **ID:** nexus
- **Description:** nexus credential

b. To create a new pipeline, go back to Dashboard, click **"New Item"** in the left navigation lane, and type the item name (i.g. `first-project`) and select **"Pipeline"**. Click **"OK"** to configure the pipeline.

c. Go to **"Pipeline"** section and select **"Pipeline script from SCM"** in the **"Definition"** field

d. Select **"Git"** in **"SCM"** field

e. Add `https://github.com/devops2021/devopsdaydayup.git` in **"Repository URL"** field

f. Select our github credential in **"Credentials"**

g. Type `*/main` in **"Branch Specifier"** field

h. Type `006-NexusJenkinsVagrantCICD/Jenkinsfile` in **"Script Path"**

i. Unselect **"Lightweight checkout"** and click "Apply" to complete the creation

j. To add maven tool, go back to **"Dashboard"** -> **"Manage Jenkins"** -> **"Global Tool Configuration"** -> Scroll down to **"Maven"** section and click **"Add Maven"**. Then fill out below fields as instructed:
**Name:** m3
**Install automaticall** selected
**Version:** 3.8.6
Click **"Save"**

### 4. Launch the Jenkins pipeline

Go to **"Dashboard"** -> Click **"first-project"** pipeline -> Click **"Build Now"**, then the Jenkins pipeline will compile the app to a war file and upload to the Nexus repository.

Login to the Nexus website (<http://0.0.0.0:8081>) and go to **"Browse"** section, and then click **"maven-nexus-repo"**, weshould be able to see the artifacts just uploaded.

![nexus-repo-configuration](images/nexus-repo-configuration.png)

### 5. Deploy a Tomcat server via Vagrant

Run below command to start up a Vagrant VM:

```dos
vagrant up
```

## 6. Download the war file and deploy to the Tomcat server

Once the deployment is done, wecan login to the Tomcat Vagrant VM and download the war from the Nexus repo. weshould be able to see the url link to download the war file in the Nexus web page. Just make sure to replace the IP address `0.0.0.0` to the actual IP of our host (running `ifconfig` to check our host IP).

![nexus-war-download-url](images/nexus-war-download-url.png)

```dos
vagrant ssh
cd /var/lib/tomcat9/webapp/
sudo wget http://<your_host_IP>:8081/repository/maven-nexus-repo/sparkjava-hello-world/sparkjava-hello-world/1.0/sparkjava-hello-world-1.0.war 
```

Wait for **2 mins** and then wecan see the war file is unzip

```dos
vagrant@vagrant:/var/lib/tomcat9/webapps$ ls
ROOT  sparkjava-hello-world-1.0  sparkjava-hello-world-1.0.war
```

Then type `exit` to exit the Vagrant VM and type below URL in yoru browser, and weshould be able to see the "Hello World" page

```dos
http://0.0.0.0:8088/sparkjava-hello-world-1.0/hello
```

![helloworld](images/helloworld.png)
