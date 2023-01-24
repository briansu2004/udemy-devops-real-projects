# Project 011: Create Read Only Kubeconfig File

Windows only

## Steps

### 1. Make sure Docker has been installed

### 2. Install kubectl

[Install and Set Up kubectl on Linux](https://www.google.com/search?channel=fs&client=ubuntu&q=install+kubectl+)

```dos
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check

sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

kubectl version --client
kubectl version --client --output=yaml
```

### 3. Install Kind

```dos
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.17.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

[The full installation guide](https://kind.sigs.k8s.io/docs/user/quick-start/))

### 4. Create a Cluster

Use below command to create a Kubernetes cluster:

```dos
kind create cluster
```

Once it is ready, you can run below command to test it out:

```dos
kubectl get node
kubectl -n default create deploy test --image=nginx
```

### 5. Create a Role ans Service Account

We will use a manifest to create a role and service account in your current context using kubectl. Please ensure you have the correct context before proceeding. You can check your current context using the following command:

```dos
kubectl config current-context
```

Then create below file as `readonly-manifest.yaml`

```dos
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

```dos
kubectl apply -f readonly-manifest.yaml 
```

<!--
> Note: As mentioned in this [ticket](https://stackoverflow.com/questions/72256006/service-account-secret-is-not-listed-how-to-fix-it), since 1.24, ServiceAccount token secrets are no longer automatically generated. (See [this note](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.24.md#urgent-upgrade-notes))
Also, the Secret is no longer used to mount credentials into Pods and you also need to manually create it. (ref: <https://kubernetes.io/docs/concepts/configuration/secret/#service-account-token-secrets>)
-->

### 6. Create ReadOnly Kubeconfig

Run the below script to generate a new readonly kubeconfig

```dos
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

Note: If you are using **AKS**, you should have a **service account** `readonly-sa` already, which has been associated with an existing readonly cluster role. You can just run below script instead:

```
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

### 5. Test

Run below command to switch to the new readonly context:

```
kubectl config use-context readonly
kubectl config current-context
```

With the new readonly kubeconfig, you can only **list**/**watch**/**exec** the Pod, but cannot **create**/**delete** any Pod. Let's test it out

```
kubectl get node
kubectl -n default get pod --watch
kubectl exec -it $(kubectl get pod --no-headers|awk '{print $1}') -- bash
```

Try creating or deleting an object:

```
kubectl delete pod $(kubectl get pod --no-headers|awk '{print $1}')
kubectl create deploy test2 --image=nginx
```

The error below indicates a permission issue, which means the kubeconfig works as expected

```
Error from server (Forbidden): pods "test-75d6d47c7f-5dshd" is forbidden: User "system:serviceaccount:default:readonly" cannot delete resource "pods" in API group "" in the namespace "default"
```

## <a name="post_project">Post Project</a>

You can delete the cluster as following command:

```
kind delete cluster
```

## <a name="troubleshooting">Troubleshooting</a>

## <a name="reference">Reference</a>

- [Read Only Kubeconfig](https://docs.hava.io/importing/kubernetes/getting-started-with-kubernetes/read-only-kubeconfig)
- [Limited Kubeconfig](https://codeforphilly.github.io/chime/operations/limited-kubeconfigs/limited-kubeconfigs.html)
