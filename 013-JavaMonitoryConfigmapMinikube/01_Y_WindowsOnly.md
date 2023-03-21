# Project Name: Develop a Java Application in K8s for Monitoring ConfigMap Modifications and Content Changes

Windows Only

## Project Goal

In this lab, you will learn how to develop a Java application that interacts with the Kubernetes API to **monitor changes** to a file that is mounted by a **ConfigMap**.

## Steps

### 1. Start Docker

Minikube needs Docker.

### 2. Start Minikube

You can install the **Minikube** by following the instruction in the [Minikube official website](https://minikube.sigs.k8s.io/docs/start/).

Once it is installed, start the minikube by running below command:

```dos
minikube start
minikube status
```

Once the Minikube starts, you can download the **kubectl** from [k8s official website](https://kubernetes.io/docs/tasks/tools/)

```dos
minikube kubectl
```

Then, when you run the command `kubectl get node`, you should see below output:

```dos
NAME       STATUS   ROLES           AGE     VERSION
minikube   Ready    control-plane   2d4h   v1.26.1
```

### 3. Build Image

Run below command to **build** the image:

```dos
git clone https://github.com/briansu2004/udemy-devops-real-projects.git
cd udemy-devops-real-projects\013-JavaMonitoryConfigmapMinikube
docker build -t java-monitor-file:v2.0 .
```

### 4. Deploy ConfigMap

```dos
kubectl apply -f configmap.yaml
```

### 5. Deploy Pod

```dos
kubectl apply -f pod.yaml
```

### 6. Verification

You can modify the contents of the ConfigMap and verify if the activity is captured in the log.

First **stream the log**

```dos
kubectl logs -f configmap-demo-pod
```

Then open another terminal to **modify the ConfigMap**

```dos
kubectl edit cm game-demo
```

**Update** anything within below **data** section

From

```dos
data:
  game.properties: "enemy.types=aliens,monsters\nplayer.maximum-lives=5\\n"
```

To

```dos
data:
  game.properties: "enemy.types=alienstest,monsters\nplayer.maximum-lives=5\\n"
```

![1679363205929](image/01_Y_WindowsOnly/1679363205929.png)

->

![1679363245390](image/01_Y_WindowsOnly/1679363245390.png)

Then wait for about 1 min and you should see below message in the log

```dos
$ kubectl logs -f configmap-demo-pod

Content has changed!
```

![1679363428176](image/01_Y_WindowsOnly/1679363428176.png)
