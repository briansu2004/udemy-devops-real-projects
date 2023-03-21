# Project 012: Backup Vault in Minio

Mac + Ubuntu (vagrant vbox)

Issue:

```bash
😄  minikube v1.29.0 on Ubuntu 20.04 (vbox/amd64)
✨  Automatically selected the docker driver. Other choices: none, ssh

⛔  Exiting due to RSRC_INSUFFICIENT_CORES: Requested cpu count 2 is greater than the available cpus of 1
```

## Project Goal

In this lab, you will deploy a helm chart with a cronjob to backup vault periodically into the Minio storage

## Steps

### 1. Install Minikube

Follow the instruction here [Minikube official website](https://minikube.sigs.k8s.io/docs/start/).

Install Minikube on Mac

```bash
curl -LO <https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64>
sudo install minikube-darwin-amd64 /usr/local/bin/minikube
```

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

```bash
DevOps 🚀 _Code % minikube start --kubernetes-version=v1.26.1
😄  minikube v1.29.0 on Darwin 13.1
✨  Automatically selected the docker driver. Other choices: virtualbox, ssh
📌  Using Docker Desktop driver with root privileges
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
    > gcr.io/k8s-minikube/kicbase...:  407.19 MiB / 407.19 MiB  100.00% 7.87 Mi
🔥  Creating docker container (CPUs=2, Memory=4000MB) ...
🐳  Preparing Kubernetes v1.26.1 on Docker 20.10.23 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔗  Configuring bridge CNI (Container Networking Interface) ...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🔎  Verifying Kubernetes components...
🌟  Enabled addons: default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

Check status:

`minikube status`

Output:

```bash
PS C:\devbox> minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

Once the Minikube starts, you can download the kubectl

```bash
minikube kubectl
```

Output:

```bash
DevOps 🚀 _Code % minikube kubectl
    > kubectl.sha256:  64 B / 64 B [-------------------------] 100.00% ? p/s 0s
    > kubectl:  51.28 MiB / 51.28 MiB [------------] 100.00% 11.58 MiB p/s 4.6s
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

```bash
NAME       STATUS   ROLES           AGE     VERSION
minikube   Ready    control-plane   4m37s   v1.25.3
```

### 3. Enable Minikube Dashboard (Optional)

You can also enable your **Minikube dashboard** by running below command:

```bash
minikube dashboard
```

You should see a Kuberentes Dashboard page pop out in your browser immediately. You can explore all Minikube resources in this UI website.

### 4. Install Helm v3.x

Follow the instruction here [Helm v3.x](https://helm.sh/docs/intro/install/)

```bash
brew install helm
```

### 5. Add Helm Repo

Once Helm is set up properly, **add** the **repo** as follows:

```bash
helm repo add minio https://charts.min.io/
```

Output:

```bash
DevOps 🚀 _Code % helm repo add minio https://charts.min.io/
"minio" has been added to your repositories
```

### 6. Create a namespace

Create a `minio` namespace

```bash
kubectl create ns minio
```

Output:

```bash
DevOps 🚀 _Code % kubectl create ns minio
namespace/minio created
```

### 7. Install Minio Helm Chart

Since we are using Minikube cluster which has only 1 node, we just deploy the Minio in a test mode.

```bash
helm install --set resources.requests.memory=512Mi --set replicas=1 --set mode=standalone --set rootUser=rootuser,rootPassword=Test1234 --generate-name minio/minio
```

Output:

```bash
DevOps 🚀 _Code % helm install --set resources.requests.memory=512Mi --set replicas=1 --set mode=standalone --set rootUser=rootuser,rootPassword=Test1234! --generate-name minio/minio    
NAME: minio-1679183256
LAST DEPLOYED: Sat Mar 18 19:47:36 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
MinIO can be accessed via port 9000 on the following DNS name from within your cluster:
minio-1679183256.default.svc.cluster.local

To access MinIO from localhost, run the below commands:

  1. export POD_NAME=$(kubectl get pods --namespace default -l "release=minio-1679183256" -o jsonpath="{.items[0].metadata.name}")

  2. kubectl port-forward $POD_NAME 9000 --namespace default

Read more about port forwarding here: http://kubernetes.io/docs/user-guide/kubectl/kubectl_port-forward/

You can now access MinIO server on http://localhost:9000. Follow the below steps to connect to MinIO server with mc client:

  1. Download the MinIO mc client - https://min.io/docs/minio/linux/reference/minio-mc.html#quickstart

  2. export MC_HOST_minio-1679183256-local=http://$(kubectl get secret --namespace default minio-1679183256 -o jsonpath="{.data.rootUser}" | base64 --decode):$(kubectl get secret --namespace default minio-1679183256 -o jsonpath="{.data.rootPassword}" | base64 --decode)@localhost:9000

  3. mc ls minio-1679183256-local

DevOps 🚀 _Code % export POD_NAME=$(kubectl get pods --namespace default -l "release=minio-1679183256" -o jsonpath="{.items[0].metadata.name}")
DevOps 🚀_Code % kubectl port-forward $POD_NAME 9000 --namespace default
Forwarding from 127.0.0.1:9000 -> 9000
Forwarding from [::1]:9000 -> 9000
```

### 8. Update the configure file

Check the minio service name and update in the `vault-backup-values.yaml` in `MINIO_ADDR` env var

```bash
MINIO_SERVICE_NAME=$(kubectl get svc -n default -o=jsonpath="{.items[1].metadata.name}")

echo Minio service name is $MINIO_SERVICE_NAME
```

Output:

```bash
DevOps 🚀 _Code % MINIO_SERVICE_NAME=$(kubectl get svc -n default -o=jsonpath="{.items[1].metadata.name}")
DevOps 🚀 _Code % echo Minio service name is $MINIO_SERVICE_NAME
Minio service name is minio-1679181453
```

Update the minio username and password in `vault-backup-values.yaml`

```bash
MINIO_USERNAME=$(kubectl get secret -l app=minio -o=jsonpath="{.items[0].data.rootUser}"|base64 -d)

echo "MINIO_USERNAME is $MINIO_USERNAME"

MINIO_PASSWORD=$(kubectl get secret -l app=minio -o=jsonpath="{.items[0].data.rootPassword}"|base64 -d)

echo "MINIO_PASSWORD is $MINIO_PASSWORD"
```

### 9. Create a Bucket in the Minio Console

In order to access the Minio console, you need to port forward it to your local

```bash
kubectl port-forward svc/$(kubectl get svc|grep console|awk '{print $1}') 9001:9001
```

Output:

```bash
DevOps 🚀 _Code % kubectl port-forward svc/$(kubectl get svc|grep console|awk '{print $1}') 9001:9001
Forwarding from 127.0.0.1:9001 -> 9001
Forwarding from [::1]:9001 -> 9001
```

Open your browser and go to this URL [http://localhost:9001](http://localhost:9001), and login with the username/password as `rootUser`/`rootPassword` setup above.

Go to *Buckets* section in the left lane and click *Create Bucket* with a name `test`, with all other setting as default.

![minio-bucket.png](images/minio-bucket.png)

### 8. Install Vault Helm Chart

We are going to deploy a Vault helm chart in the Minikube cluster. Create a `vault-values.yaml` first:

```bash
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

```bash
helm repo add hashicorp https://helm.releases.hashicorp.com
kubectl create ns vault-test
helm -n vault-test install vault hashicorp/vault -f vault-values.yaml
```

Initiate the Vault

```bash
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

```bash
kubectl -n vault-test create configmap upload --from-file=upload.sh
helm -n vault-test upgrade --install vault-backup helm-chart -f vault-backup-values.yaml
kubectl -n vault-test create job vault-backup-test --from=cronjob/vault-backup-cronjob
```

### 10. Verification

Port forward Minio console to your local host:

```bash
MINIO_CONSOLE_ADDR=$(kubectl -n minio get svc|grep console|awk '{print $1}')
kubectl -n minio port-forward svc/$MINIO_CONSOLE_ADDR 9001:9001
```

Login to the Minio console [http://localhost:9001](http://localhost:9001) and go to **Object Browser** section in the left navigation lane. Click **Test** bucket and you should see the backup files list there.
![minio-console.png](images/minio-console.png)

<!--
Reference

[Minio Helm Deployment](https://github.com/minio/minio/tree/master/helm/minio)
-->
