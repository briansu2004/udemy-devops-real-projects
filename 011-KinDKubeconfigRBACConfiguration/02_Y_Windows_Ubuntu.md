# Lab 011: Create Read Only Kubeconfig File

Windows + Ubuntu

## Prerequisites

### 1. Install Vagrant

## Steps

### 1. Install Docker in Ubuntu

### 2. Install kubectl

<!--
[Install and Set Up kubectl on Linux](https://www.google.com/search?channel=fs&client=ubuntu&q=install+kubectl+)
-->

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check

sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

kubectl version --client --output=yaml
```

### 3. Install Kind

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.17.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

<!--
[The full installation guide](https://kind.sigs.k8s.io/docs/user/quick-start/)
-->

### 4. Create a Cluster

Use below command to create a Kubernetes cluster:

```bash
kind create cluster
```

<!--
```bash
vagrant@vagrant:~$ kind create cluster
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.25.3) 🖼 
 ✓ Preparing nodes 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Have a nice day! 👋
vagrant@vagrant:~$ 
```
-->

Once it is ready, you can run below command to test it out:

```bash
kubectl get node
kubectl -n default create deploy test --image=nginx
```

<!--
```bash
vagrant@vagrant:~$ kubectl get node
NAME                 STATUS   ROLES           AGE   VERSION
kind-control-plane   Ready    control-plane   53s   v1.25.3
vagrant@vagrant:~$ 
vagrant@vagrant:~$
vagrant@vagrant:~$ kubectl -n default create deploy test --image=nginx
deployment.apps/test created
```
-->

### 5. Create a Role ans Service Account

We will use a manifest to create a role and service account in our current context using kubectl.

Please make sure to have the correct context before proceeding.

We can check our current context using the following command:

```bash
kubectl config current-context
```

<!--
```bash
vagrant@vagrant:~$ kubectl config current-context
kind-kind
```
-->

Then create below file as `readonly-manifest.yaml`

```bash
cat > ~/readonly-manifest.yaml <<EOF

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: readonly
  namespace: default
---
apiVersion: v1
kind: Secret
metadata:
  name: readonly-token
  annotations:
    kubernetes.io/service-account.name: readonly
type: kubernetes.io/service-account-token
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
  name: readonly-clusterrole
  namespace: default
rules:
  - apiGroups:
      - ""
    resources: ["*"]
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - ""
    resources: 
      - pods/exec
      - pods/portforward
    verbs:
      - create
  - apiGroups:
      - extensions
    resources: ["*"]
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - apps
    resources: ["*"]
    verbs:
      - get
      - list
      - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: readonly-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: readonly-clusterrole
subjects:
  - kind: ServiceAccount
    name: readonly
    namespace: default
EOF
```

Apply the manifest file:

```bash
kubectl apply -f readonly-manifest.yaml 
```

<!--
```bash
vagrant@vagrant:~$ kubectl apply -f readonly-manifest.yaml 
serviceaccount/readonly created
secret/readonly-token created
clusterrole.rbac.authorization.k8s.io/readonly-clusterrole created
clusterrolebinding.rbac.authorization.k8s.io/readonly-binding created
```
-->

<!--
> Note: As mentioned in this [ticket](https://stackoverflow.com/questions/72256006/service-account-secret-is-not-listed-how-to-fix-it), since 1.24, ServiceAccount token secrets are no longer automatically generated. (See [this note](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.24.md#urgent-upgrade-notes))
Also, the Secret is no longer used to mount credentials into Pods and you also need to manually create it. (ref: <https://kubernetes.io/docs/concepts/configuration/secret/#service-account-token-secrets>)
-->

### 6. Create ReadOnly Kubeconfig

Run the below script to generate a new readonly kubeconfig

```bash
server=$(kubectl config view --minify --output jsonpath='{.clusters[*].cluster.server}')
ca=$(kubectl get secret/readonly-token --namespace=default -o jsonpath='{.data.ca\.crt}')
token=$(kubectl get secret/readonly-token --namespace=default -o jsonpath='{.data.token}' | base64 --decode)
namespace=$(kubectl get secret/readonly-token --namespace=default -o jsonpath='{.data.namespace}' | base64 --decode)

echo "
apiVersion: v1
kind: Config
clusters:
- name: kind-readonly
  cluster:
    certificate-authority-data: ${ca}
    server: ${server}
contexts:
- name: readonly
  context:
    cluster: kind-readonly
    namespace: default
    user: readonly-user
current-context: readonly
users:
- name: readonly-user
  user:
    token: ${token}
" > readonly-kubeconfig.yml

cp ~/.kube/config ~/.kube/config.$(date +%Y%m%d).bak
KUBECONFIG=~/.kube/config:readonly-kubeconfig.yml kubectl config view --flatten > /tmp/merged-config
mv /tmp/merged-config ~/.kube/config
```

<!--
```bash
vagrant@vagrant:~$ echo $server
https://127.0.0.1:33157
vagrant@vagrant:~$ echo $ca
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMvakNDQWVhZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJek1EUXhNakF3TURjd05Wb1hEVE16TURRd09UQXdNRGN3TlZvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTVZZCmVpWnQvQmh5UEc2dURiQ2xYd3RKYjV2eE1WMUNIVWpEdDRnMC83d01yUEIwVTNXQWE4Q09YSlROWjUxNXRuQlEKMmV1N1BHTU0rVHRSajBVeCtpWFdkTHp4L0R5SFMvcDk1djFkUVJ4YmFQZlFNeW84SklGZDc0bEYxN0V3ZXpDUQpndTJrVkdENXQzMVdEQkVpM0pEenZDMEhlc1dTRDVGQkhvZ3poT2t3THZRSzNGSm0zNzhSZnNuTGlja2ErUm5MCjdPZ1pRZDhWOEFqd3lYOWZtL2JIZWw1VE90UnAzOVhGZmtqNXBVbzdNODAvbVczemgvMEVxT1ZvWDhqYmY1VkUKMVFYSUxLSXdKK052Mk96TWM0ZkRYZ2R4MFdKR3oyMzAxYzZxeVJxMy8xZ2piRks4aXBJcDFPK25BdzZDZUlyZQpvRGM0MUFkWFFSWkN5a0VCYmRVQ0F3RUFBYU5aTUZjd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZDTGVabGZLTCs0UkcwaDRIcjQ2Sy8rbzlNd2xNQlVHQTFVZEVRUU8KTUF5Q0NtdDFZbVZ5Ym1WMFpYTXdEUVlKS29aSWh2Y05BUUVMQlFBRGdnRUJBQ1RhQXlqb1ZBSFM3OURoRDdwQQphRUt5R1JNb3NhQ2FvS3kwL0lmV0Q0UVJuS2dKR0hDbTRDWFhtdUovREMzaVZ2UzVva0xNSi9QV2hyVHFPQXJECjBjd05FYUR4VTNxcGg5M0haekNIVFJzZnlURFAzdlVrRTl6K1NMcVVIczR0T2hobXhJVUhkb3NhMHpsODFuUk4KaWR0RlVjeVBQR0dBeGxPZGlMdHJKcnF1NWZ0ZEkwNC9rMGNvOFBuZDh6d1ZPNUZ0YVFVS3QrNXdkMVpKZEh1UwpOUzRTYWxtdGZMb0l2MUtpRVBzbVI3SG1SbUgwUFdHdml1em91dmF5TVk5d3BMOVJ6dDE3QlRtS1lUTlZaS3AwCmkwVFNTT2xuVC9jNFJDcnl6VlNYcFROemlEVnNEZWV3a0ZPWnVoRlA0SklFMEpNT3hhSXlDSGxadU5HTmtSQ28KcVJJPQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
vagrant@vagrant:~$ echo $token
eyJhbGciOiJSUzI1NiIsImtpZCI6InRUTmd4d2thVTlUWWxrSGx5U1pLcEFRejhBa21DdDNZdnZuZ3FEUGVaUEUifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6InJlYWRvbmx5LXRva2VuIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6InJlYWRvbmx5Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiNjAxNjdjYWMtYWY1OS00ZTAxLWI1MGYtZmQ0NjYyODUwMzI0Iiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OmRlZmF1bHQ6cmVhZG9ubHkifQ.BIjW4TQOqrzae7c1dsInvjk0Xzib3x-ijOozc48Y9_grucmyw6E05iGo0Rz-KpbWPu_qrV-NzC2AlWXf1UtP2BDCExhkF9Q5exUPLX_NFu1pnmP5fMtEyEfCOX789C_t2xmbdLAMp3XCsP3ux5tcf4m71R9HW4SFVVF82R_uOkYGkUKpacS5Y1NOyWRm87u33PV7JL7vXD71HGiiW-epgvUZfl-aStFG8gsm-xXdKP8f14cbSu4bE-gPHAvynklMfHqTbLpH3eSzbEoCTgDhj_GtOVK42ovQKI-t_SoboARsqbv9ore9fGHzEfqJvmzctl_3drQYzBiYe1Mmdcs-Rg
vagrant@vagrant:~$ echo $namespace
default

vagrant@vagrant:~$ KUBECONFIG=~/.kube/config:readonly-kubeconfig.yml kubectl config view --flatten > /tmp/merged-config
vagrant@vagrant:~$
vagrant@vagrant:~$ echo $KUBECONFIG
/home/vagrant/.kube/config:readonly-kubeconfig.yml
vagrant@vagrant:~$
vagrant@vagrant:~$ mv /tmp/merged-config ~/.kube/config
```
-->

<!--
Note: If you are using **AKS**, you should have a **service account** `readonly-sa` already, which has been associated with an existing readonly cluster role. You can just run below script instead:

```bash
api_server=$(kubectl config view -o jsonpath='{.clusters[0].cluster.server}')
cluster_name=$(kubectl config view -o jsonpath='{.clusters[0].name}')
serviceaccount_name=$(kubectl -n default get serviceaccount/readonly-sa -o jsonpath='{.secrets[0].name}')
ca=$(kubectl -n default get secret/$serviceaccount_name -o jsonpath='{.data.ca\.crt}')
token=$(kubectl -n default get secret/$serviceaccount_name -o jsonpath='{.data.token}' | base64 --decode)
namespace=$(kubectl -n default get secret/$serviceaccount_name -o jsonpath='{.data.namespace}' | base64 --decode)

echo "
apiVersion: v1
kind: Config
clusters:
- name: ${cluster_name}
  cluster:
    certificate-authority-data: ${ca}
    server: ${api_server}
contexts:
- name: ${cluster_name}
  context:
    cluster: ${cluster_name}
    namespace: default
    user: readonly-user
current-context: ${cluster_name}
users:
- name: readonly-user
  user:
    token: ${token}
" > readonly.config

cp ~/.kube/config ~/.kube/config.$(date +%Y%m%d).bak
KUBECONFIG=~/.kube/config:readonly.config kubectl config view --flatten > /tmp/merged-config
mv /tmp/merged-config ~/.kube/config
```
-->

### 7. Test

Switch to the new readonly context:

```bash
kubectl config use-context readonly
kubectl config current-context
```

<!--
```bash
vagrant@vagrant:~$ kubectl config use-context readonly
Switched to context "readonly".
vagrant@vagrant:~$
vagrant@vagrant:~$ kubectl config current-context
readonly
```
-->

With the new readonly kubeconfig, we can only **list**/**watch**/**exec** the Pod, but can not **create**/**delete** any Pod.

Let's test it out:

```bash
kubectl get node

kubectl -n default get pod --watch

kubectl exec -it $(kubectl get pod --no-headers|awk '{print $1}') -- bash
```

Try creating or deleting an object:

```bash
kubectl delete pod $(kubectl get pod --no-headers|awk '{print $1}')

kubectl create deploy test2 --image=nginx
```

<!--
```bash
vagrant@vagrant:~$ kubectl delete pod $(kubectl get pod --no-headers|awk '{print $1}')
Error from server (Forbidden): pods "test-75d6d47c7f-xbq79" is forbidden: User "system:serviceaccount:default:readonly" cannot delete resource "pods" in API group "" in the namespace "default"
vagrant@vagrant:~$ 
vagrant@vagrant:~$ kubectl create deploy test2 --image=nginx
error: failed to create deployment: deployments.apps is forbidden: User "system:serviceaccount:default:readonly" cannot create resource "deployments" in API group "apps" in the namespace "default"
```
-->

The error below indicates a permission issue, which means the kubeconfig works as expected

```bash
Error from server (Forbidden): pods "test-75d6d47c7f-5dshd" is forbidden: User "system:serviceaccount:default:readonly" cannot delete resource "pods" in API group "" in the namespace "default"
```
