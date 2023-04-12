# Lab 011: Create Read Only Kubeconfig File

Windows only

## Prerequisites

### Install Docker in Windows

### Install Git Bash in Windows

## Steps

### 1. Install kubectl in Windows

<!--
[Install and Set Up kubectl on Linux](https://www.google.com/search?channel=fs&client=ubuntu&q=install+kubectl+)
-->

```dos
choco install kubernetes-cli
```

### 2. Install Kind

```dos
choco install kind
```

<!--
[The full installation guide](https://kind.sigs.k8s.io/docs/user/quick-start/)
-->

### 3. Create a Cluster

Use below command to create a Kubernetes cluster:

```dos
kind create cluster
```

<!--
```dos
C:\devbox>kind create cluster
Creating cluster "kind" ...
 • Ensuring node image (kindest/node:v1.26.3) 🖼  ...
 ✓ Ensuring node image (kindest/node:v1.26.3) 🖼
 • Preparing nodes 📦   ...
 ✓ Preparing nodes 📦 
 • Writing configuration 📜  ...
 ✓ Writing configuration 📜
 • Starting control-plane 🕹️  ...
 ✓ Starting control-plane 🕹️
 • Installing CNI 🔌  ...
 ✓ Installing CNI 🔌
 • Installing StorageClass 💾  ...
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Not sure what to do next? 😅  Check out https://kind.sigs.k8s.io/docs/user/quick-start/
```
-->

Once it is ready, you can run below command to test it out:

```dos
kubectl get node

kubectl -n default create deploy test --image=nginx
```

<!--
```dos
C:\devbox>kubectl get node
NAME                 STATUS   ROLES           AGE   VERSION
kind-control-plane   Ready    control-plane   44s   v1.26.3

C:\devbox>kubectl -n default create deploy test --image=nginx
deployment.apps/test created

PS C:\devbox> kubectl get deployment
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
test   0/1     1            0           9s

PS C:\devbox> kubectl get pod
NAME                    READY   STATUS              RESTARTS   AGE
test-865bcfc74c-zqwdq   0/1     ContainerCreating   0          5s
```
-->

### 4. Create a Role and a Service Account

We will use a manifest to create a role and service account in our current context using kubectl.

Please make sure to have the correct context before proceeding.

We can check our current context using the following command:

```dos
kubectl config current-context
```

<!--
```dos
vagrant@vagrant:~$ kubectl config current-context
kind-kind
```
-->

Then create below file as `readonly-manifest.yaml`

```dos
cat > readonly-manifest.yaml <<EOF

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

```dos
kubectl apply -f readonly-manifest.yaml 
```

<!--
```dos
vagrant@vagrant:~$ kubectl apply -f readonly-manifest.yaml 
serviceaccount/readonly created
secret/readonly-token created
clusterrole.rbac.authorization.k8s.io/readonly-clusterrole created
clusterrolebinding.rbac.authorization.k8s.io/readonly-binding created

PS C:\devbox> kubectl get ServiceAccount
NAME       SECRETS   AGE
default    0         2m6s
readonly   0         11s
PS C:\devbox> kubectl get ClusterRole | findstr readonly
readonly-clusterrole                                                   2023-04-12T20:43:02Z

PS C:\devbox> kubectl get ClusterRoleBinding | findstr readonly
readonly-binding                                       ClusterRole/readonly-clusterrole                                                   90s
```
-->

<!--
> Note: As mentioned in this [ticket](https://stackoverflow.com/questions/72256006/service-account-secret-is-not-listed-how-to-fix-it), since 1.24, ServiceAccount token secrets are no longer automatically generated. (See [this note](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.24.md#urgent-upgrade-notes))
Also, the Secret is no longer used to mount credentials into Pods and you also need to manually create it. (ref: <https://kubernetes.io/docs/concepts/configuration/secret/#service-account-token-secrets>)
-->

### 5. Create ReadOnly Kubeconfig

Run the below script to generate a new readonly kubeconfig

```dos
server=$(kubectl config view --minify --output jsonpath='{.clusters[*].cluster.server}')
ca=$(kubectl get secret/readonly-token --namespace=default -o jsonpath='{.data.ca\.crt}')
token=$(kubectl get secret/readonly-token --namespace=default -o jsonpath='{.data.token}' | base64 --decode)
namespace=$(kubectl get secret/readonly-token --namespace=default -o jsonpath='{.data.namespace}' | base64 --decode)

echo $server
echo $ca
echo $token
echo $namespace

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

KUBECONFIG=~/.kube/config:"/c/devbox/readonly-kubeconfig.yml" kubectl config view --flatten > /tmp/merged-config

mv /tmp/merged-config ~/.kube/config
```

<!--
Lessons learned:

Clean up ~/.kube if needed

Compare `~/.kube/config` and the back up file to see the differences.

Has to be 1 line!!!

`KUBECONFIG=~/.kube/config:"/c/devbox/readonly-kubeconfig.yml" kubectl config view --flatten > /tmp/merged-config`

After it, compare config and the backup to see the differences.

```bash
kubectl config view

kubectl config get-contexts

kubectl get pods --namespace <namespace>
```
-->

### 6. Test

Switch to the new readonly context:

```dos
kubectl config use-context readonly

kubectl config current-context
```

<!--
```dos
vagrant@vagrant:~$ kubectl config use-context readonly
Switched to context "readonly".
vagrant@vagrant:~$
vagrant@vagrant:~$ kubectl config current-context
readonly
```
-->

With the new readonly kubeconfig, we can only **list**/**watch**/**exec** the Pod, but can not **create**/**delete** any Pod.

Let's try:

```bash
kubectl get node

kubectl -n default get pod --watch

kubectl exec -it $(kubectl get pod --no-headers|awk '{print $1}') -- script --quiet --return --command "bash"
exit
```

Try to delete an existing object:

```bash
kubectl delete pod $(kubectl get pod --no-headers|awk '{print $1}')
```

The error below indicates a permission issue, which means the kubeconfig works as expected

```dos
Error from server (Forbidden): pods "test-75d6d47c7f-xbq79" is forbidden: User "system:serviceaccount:default:readonly" cannot delete resource "pods" in API group "" in the namespace "default"
```

Try to create a new object:

```bash
kubectl create deploy test2 --image=nginx
```

The error below indicates a permission issue, which means the kubeconfig works as expected

```dos
error: failed to create deployment: deployments.apps is forbidden: User "system:serviceaccount:default:readonly" cannot create resource "deployments" in API group "apps" in the namespace "default"
```
