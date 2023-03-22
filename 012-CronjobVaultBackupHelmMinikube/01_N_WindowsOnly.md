# Project 012: Backup Vault in Minio

Windows Only

For 1 CPU Windows, `Windows + Ubuntu (vagrant vbox)` doesn't work - It works with Docker.

However, the Minio has issues.

## Project Goal

In this lab, you will deploy a helm chart with a cronjob to backup vault periodically into the Minio storage

## Steps

### 1. Install and run Docker for Windows

...

### 1. Install Minikube for Windows

Follow the instruction here [Minikube official website](https://minikube.sigs.k8s.io/docs/start/).

### 2. Start Minikube

Once it is installed, start the minikube by running below command:

`minikube start`

or

`minikube start --kubernetes-version=v1.25.3`

or

`minikube start --kubernetes-version=v1.26.1`

<!--
This one doesn't work -

`minikube start --cpus=1`
-->

Output:

```dos
PS C:\devbox> minikube start --kubernetes-version=v1.26.1
* minikube v1.29.0 on Microsoft Windows 10 Enterprise 10.0.19044.2604 Build 19044.2604
* Using the docker driver based on existing profile
* Starting control plane node minikube in cluster minikube
* Pulling base image ...
* Updating the running docker "minikube" container ...
* Preparing Kubernetes v1.26.1 on Docker 20.10.23 ...
* Configuring bridge CNI (Container Networking Interface) ...
* Verifying Kubernetes components...
  - Using image gcr.io/k8s-minikube/storage-provisioner:v5
* Enabled addons: storage-provisioner, default-storageclass
* Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

<!--
PS C:\devbox> minikube start
* minikube v1.29.0 on Microsoft Windows 10 Enterprise 10.0.19044.2604 Build 19044.2604
* Using the docker driver based on existing profile
* Starting control plane node minikube in cluster minikube
* Pulling base image ...
* Restarting existing docker container for "minikube" ...
* Preparing Kubernetes v1.26.1 on Docker 20.10.23 ...
* Configuring bridge CNI (Container Networking Interface) ...
* Verifying Kubernetes components...
  - Using image docker.io/kubernetesui/dashboard:v2.7.0
  - Using image docker.io/kubernetesui/metrics-scraper:v1.0.8
  - Using image gcr.io/k8s-minikube/storage-provisioner:v5
* Some dashboard features require the metrics-server addon. To enable all features please run:

        minikube addons enable metrics-server

* Enabled addons: storage-provisioner, default-storageclass, dashboard
* Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
PS C:\devbox> minikube addons enable metrics-server
* metrics-server is an addon maintained by Kubernetes. For any concerns contact minikube on GitHub.
You can view the list of minikube maintainers at: https://github.com/kubernetes/minikube/blob/master/OWNERS
  - Using image registry.k8s.io/metrics-server/metrics-server:v0.6.2
* The 'metrics-server' addon is enabled
PS C:\devbox> minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured

-->

Check status:

`minikube status`

Output:

```dos
PS C:\devbox> minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

Once the Minikube starts, you can download the kubectl

```dos
minikube kubectl
```

Output:

```dos
PS C:\devbox> minikube kubectl
kubectl controls the Kubernetes cluster manager.

 Find more information at: https://kubernetes.io/docs/reference/kubectl/

Basic Commands (Beginner):
  create          Create a resource from a file or from stdin
  expose          Take a replication controller, service, deployment or pod and expose it as a new Kubernetes service
  run             Run a particular image on the cluster
  set             Set specific features on objects
...
```

Then, when you run the command `kubectl get node`, you should see below output:

```dos
NAME       STATUS   ROLES           AGE     VERSION
minikube   Ready    control-plane   4m37s   v1.25.3
```

### 3. Enable Minikube Dashboard (Optional)

You can also enable your **Minikube dashboard** by running below command:

```dos
minikube dashboard
```

You should see a Kuberentes Dashboard page pop out in your browser immediately. You can explore all Minikube resources in this UI website.

### 4. Install Helm v3.x

Follow the instruction here [Helm v3.x](https://helm.sh/docs/intro/install/)

```dos
choco install kubernetes-helm
```

### 5. Add Helm Repo

Once Helm is set up properly, **add** the **repo** as follows:

```dos
helm repo add minio https://charts.min.io/
```

### 6. Create a namespace

Create a `minio` namespace

```dos
kubectl create ns minio
```

Output:

```dos
PS C:\devbox> kubectl create ns minio
namespace/minio created
```

### 7. Install Minio Helm Chart

Since we are using Minikube cluster which has only 1 node, we just deploy the Minio in a test mode.

```dos
helm install --set resources.requests.memory=512Mi --set replicas=1 --set mode=standalone --set rootUser=rootuser,rootPassword=Test1234! --generate-name minio/minio
```

or

```dos
helm install --set resources.requests.memory=512Mi --set replicas=1 --set mode=standalone --set rootUser=rootuser,rootPassword=Test1234 --generate-name minio/minio
```

Output:

```dos
PS C:\devbox> helm install --set resources.requests.memory=512Mi --set replicas=1 --set mode=standalone --set rootUser=rootuser,rootPassword=Test1234! --generate-name minio/minio
NAME: minio-1679172101
LAST DEPLOYED: Sat Mar 18 16:41:42 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
MinIO can be accessed via port 9000 on the following DNS name from within your cluster:
minio-1679172101.default.svc.cluster.local

To access MinIO from localhost, run the below commands:

  1. export POD_NAME=$(kubectl get pods --namespace default -l "release=minio-1679172101" -o jsonpath="{.items[0].metadata.name}")

  2. kubectl port-forward $POD_NAME 9000 --namespace default

Read more about port forwarding here: http://kubernetes.io/docs/user-guide/kubectl/kubectl_port-forward/

You can now access MinIO server on http://localhost:9000. Follow the below steps to connect to MinIO server with mc client:

  1. Download the MinIO mc client - https://min.io/docs/minio/linux/reference/minio-mc.html#quickstart

  2. export MC_HOST_minio-1679172101-local=http://$(kubectl get secret --namespace default minio-1679172101 -o jsonpath="{.data.rootUser}" | base64 --decode):$(kubectl get secret --namespace default minio-1679172101 -o jsonpath="{.data.rootPassword}" | base64 --decode)@localhost:9000

  3. mc ls minio-1679172101-local
```

<!--
PS C:\devbox> helm list
NAME                    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
minio-1679175880        default         1               2023-03-18 17:44:41.4395894 -0400 EDT   deployed        minio-5.0.7     RELEASE.2023-02-10T18-48-39Z
PS C:\devbox> helm uninstall  minio-5.0.7
Error: uninstall: Release not loaded: minio-5.0.7: release: not found

PS C:\devbox> helm uninstall minio-1679175880
release "minio-1679175880" uninstalled

PS C:\devbox> helm install --set resources.requests.memory=512Mi --set replicas=1 --set mode=standalone --set rootUser=rootuser,rootPassword=Test1234! --generate-name minio/minio
NAME: minio-1679439883
LAST DEPLOYED: Tue Mar 21 19:04:44 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
MinIO can be accessed via port 9000 on the following DNS name from within your cluster:
minio-1679439883.default.svc.cluster.local

To access MinIO from localhost, run the below commands:

  1. export POD_NAME=$(kubectl get pods --namespace default -l "release=minio-1679439883" -o jsonpath="{.items[0].metadata.name}")

  2. kubectl port-forward $POD_NAME 9000 --namespace default

Read more about port forwarding here: http://kubernetes.io/docs/user-guide/kubectl/kubectl_port-forward/

You can now access MinIO server on http://localhost:9000. Follow the below steps to connect to MinIO server with mc client:

  1. Download the MinIO mc client - https://min.io/docs/minio/linux/reference/minio-mc.html#quickstart

  2. export MC_HOST_minio-1679439883-local=http://$(kubectl get secret --namespace default minio-1679439883 -o jsonpath="{.data.rootUser}" | base64 --decode):$(kubectl get secret --namespace default minio-1679439883 -o jsonpath="{.data.rootPassword}" | base64 --decode)@localhost:9000

  3. mc ls minio-1679439883-local
-->

### 8. Update the configure file

Check the minio service name and update in the `vault-backup-values.yaml` in `MINIO_ADDR` env var

```dos
$POD_NAME = kubectl get pods --namespace default -l "release=minio-1679439883" -o jsonpath="{.items[0].metadata.name}"
echo "Minio POD name is $POD_NAME"
kubectl port-forward $POD_NAME 9000 --namespace default

$MINIO_SERVICE_NAME = kubectl get svc -o jsonpath="{.items[1].metadata.name}"
echo "Minio service name is $MINIO_SERVICE_NAME"
```

Output:

```dos
PS C:\devbox> $POD_NAME = kubectl get pods --namespace default -l "release=minio-1679442603" -o jsonpath="{.items[0].metadata.name}"
PS C:\devbox>
PS C:\devbox> echo "Minio POD name is $POD_NAME"
Minio POD name is minio-1679175880-74bc6487b8-lqmqx
PS C:\devbox>
PS C:\devbox> kubectl port-forward $POD_NAME 9000 --namespace default
Forwarding from 127.0.0.1:9000 -> 9000
Forwarding from [::1]:9000 -> 9000
```

<!--
PS C:\devbox> kubectl get pods
NAME                              READY   STATUS    RESTARTS       AGE
configmap-demo-pod                1/1     Running   1 (9m4s ago)   21h
minio-1679439883-dd748c6c-n74np   1/1     Running   0              2m52s

PS C:\devbox> $POD_NAME = kubectl get pods --namespace default -l "release=minio-1679439883" -o jsonpath="{.items[0].metadata.name}"
PS C:\devbox> echo "Minio POD name is $POD_NAME"
Minio POD name is minio-1679439883-dd748c6c-n74np

PS C:\devbox>  kubectl get svc
NAME                       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
kubernetes                 ClusterIP   10.96.0.1       <none>        443/TCP    3d1h
minio-1679439883           ClusterIP   10.101.97.206   <none>        9000/TCP   3m57s
minio-1679439883-console   ClusterIP   10.107.132.79   <none>        9001/TCP   3m57s

PS C:\devbox> $MINIO_SERVICE_NAME = kubectl get svc -o jsonpath="{.items[1].metadata.name}"
PS C:\devbox> echo "Minio service name is $MINIO_SERVICE_NAME"
Minio service name is minio-1679439883
```
-->

Update the minio username and password in `vault-backup-values.yaml`

<!--
```dos
MINIO_USERNAME=$(kubectl get secret -l app=minio -o=jsonpath="{.items[0].data.rootUser}"|base64 -d)
echo "MINIO_USERNAME is $MINIO_USERNAME"
MINIO_PASSWORD=$(kubectl get secret -l app=minio -o=jsonpath="{.items[0].data.rootPassword}"|base64 -d)
echo "MINIO_PASSWORD is $MINIO_PASSWORD"
```
-->

```dos
$MINIO_USERNAME = kubectl get secret -l app=minio -o=jsonpath="{.items[0].data.rootUser}"

echo "MINIO_USERNAME is $MINIO_USERNAME"

$MINIO_PASSWORD = kubectl get secret -l app=minio -o=jsonpath="{.items[0].data.rootPassword}"

echo "MINIO_PASSWORD is $MINIO_PASSWORD"
```

### 9. Create a Bucket in the Minio Console

In order to access the Minio console, you need to port forward it to your local

<!--
kubectl port-forward svc/$(kubectl get svc|grep console|awk '{print $1}') 9001:9001
-->

```dos
kubectl get svc | findstr console
kubectl port-forward svc/minio-1679439883-console 9001:9001


kubectl port-forward svc/minio-1679442603-console 9001:9001
```

<!--
PS C:\devbox> kubectl get svc | findstr console
minio-1679439883-console   ClusterIP   10.107.132.79   <none>        9001/TCP   6m56s
PS C:\devbox> kubectl port-forward svc/minio-1679439883-console 9001:9001
Forwarding from 127.0.0.1:9001 -> 9001
Forwarding from [::1]:9001 -> 9001

-->

Output:

```dos
PS C:\devbox> kubectl get svc | findstr console
minio-1679172101-console   ClusterIP   10.107.12.117   <none>        9001/TCP   24m

PS C:\devbox> kubectl port-forward svc/minio-1679172101-console 9001:9001
Forwarding from 127.0.0.1:9001 -> 9001
Forwarding from [::1]:9001 -> 9001
```

Open your browser and go to this URL [http://localhost:9001](http://localhost:9001), and login with the username/password as `rootUser`/`rootPassword` setup above.

Go to *Buckets* section in the left lane and click *Create Bucket* with a name `test`, with all other setting as default.

![minio-bucket.png](images/minio-bucket.png)

### 8. Install Vault Helm Chart

We are going to deploy a Vault helm chart in the Minikube cluster. Create a `vault-values.yaml` first:

```dos
cat <<EOF > vault-values.yaml

injector:
  enabled: "-"
  replicas: 1
  image:
    repository: "hashicorp/vault-k8s"
    tag: "1.1.0"

server:
  enabled: "-"
  image:
    repository: "hashicorp/vault"
    tag: "1.12.1"
EOF
```

Run below commands to apply the helm chart:

```dos
helm repo add hashicorp https://helm.releases.hashicorp.com
kubectl create ns vault-test
helm -n vault-test install vault hashicorp/vault -f vault-values.yaml
```

Initiate the Vault

```dos
kubectl -n vault-test exec vault-0 -- vault operator init
kubectl -n vault-test exec vault-0 -- vault operator unseal <unseal key from .vault.key>
kubectl -n vault-test exec vault-0 -- vault operator unseal <unseal key from .vault.key>
kubectl -n vault-test exec vault-0 -- vault operator unseal <unseal key from .vault.key>

# Enable secrets engine and import config
kubectl -n vault-test exec vault-0 -- vault login <root token>
kubectl -n vault-test exec vault-0 -- vault secrets enable -version=2 kv-v2 
kubectl -n vault-test exec vault-0 -- vault kv put -mount=kv-v2 devops-secret username=root password=changeme

# Enable approle engine and create role/secret
kubectl -n vault-test exec -it vault-0 -- sh

cd /tmp
cat > policy.hcl  <<EOF
path "kv-v2/*" {
  capabilities = ["create", "update","read"]
}
EOF
vault policy write first-policy policy.hcl
vault policy list
vault policy read first-policy

vault auth enable approle
vault write auth/approle/role/first-role \
    secret_id_ttl=10000m \
    token_num_uses=10 \
    token_ttl=20000m \
    token_max_ttl=30000m \
    secret_id_num_uses=40 \
    token_policies=first-policy
export ROLE_ID="$(vault read -field=role_id auth/approle/role/first-role/role-id)"
echo Role_ID is $ROLE_ID
export SECRET_ID="$(vault write -f -field=secret_id auth/approle/role/first-role/secret-id)"
echo SECRET_ID is $SECRET_ID
```

### 9. Deploy Vault Backup Helm Chart

```dos
kubectl -n vault-test create configmap upload --from-file=upload.sh
helm -n vault-test upgrade --install vault-backup helm-chart -f vault-backup-values.yaml
kubectl -n vault-test create job vault-backup-test --from=cronjob/vault-backup-cronjob
```

### 10. Verification

Port forward Minio console to your local host:

```dos
MINIO_CONSOLE_ADDR=$(kubectl -n minio get svc|grep console|awk '{print $1}')
kubectl -n minio port-forward svc/$MINIO_CONSOLE_ADDR 9001:9001
```

Login to the Minio console [http://localhost:9001](http://localhost:9001) and go to **Object Browser** section in the left navigation lane. Click **Test** bucket and you should see the backup files list there.
![minio-console.png](images/minio-console.png)

<!--
Reference

[Minio Helm Deployment](https://github.com/minio/minio/tree/master/helm/minio)
-->
