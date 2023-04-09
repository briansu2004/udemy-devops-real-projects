# Project 014: Helm Deployment in Kubernetes

Windows

Issues:

`vault operator init` has issues.

<!--
```dos
PS C:\devbox\udemy-devops-real-projects\014-VaultInjectorMinikube> vault operator init
vault : The term 'vault' is not recognized as the name of a cmdlet, function, script file, or operable program. Check the spelling of the name, or if a path
was included, verify that the path is correct and try again.
At line:1 char:1
+ vault operator init
+ ~~~~~
    + CategoryInfo          : ObjectNotFound: (vault:String) [], CommandNotFoundException
    + FullyQualifiedErrorId : CommandNotFoundException
```
-->

```dos
PS C:\devbox\udemy-devops-real-projects\014-VaultInjectorMinikube> docker ps
CONTAINER ID   IMAGE                                 COMMAND                  CREATED      STATUS         PORTS                                                                                                                             NAMES
c12c356d0ba7   gcr.io/k8s-minikube/kicbase:v0.0.37   "/usr/local/bin/entr…"   6 days ago   Up 4 minutes   127.0.0.1:2233->22/tcp, 127.0.0.1:2234->2376/tcp, 127.0.0.1:2236->5000/tcp, 127.0.0.1:2237->8443/tcp, 127.0.0.1:2235->32443/tcp   minikube
PS C:\devbox\udemy-devops-real-projects\014-VaultInjectorMinikube>
PS C:\devbox\udemy-devops-real-projects\014-VaultInjectorMinikube> docker exec -it c12 bash
root@minikube:/#
root@minikube:/# vault operator init
bash: vault: command not found
```

## Project Goal

In this lab, we will learn how to deploy a Jenkins via Helm Chart in Kubernetes.

## Prerequisites

### 1. Install Docker for Windows

### 2. Install Minikube for Windows

```dos
minikube start
```

or

```dos
minikube start --kubernetes-version=v1.26.1
```

### 3. Install Helm for Windows

## Steps

### 1. Add Helm Repo

To use the Helm chart, **add the Hashicorp helm repository** and check that you have access to the chart:

```dos
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update
helm search repo hashicorp/vault
```

<!--
PS C:\devbox> helm repo add hashicorp https://helm.releases.hashicorp.com
"hashicorp" has been added to your repositories
PS C:\devbox> helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "hashicorp" chart repository
...Successfully got an update from the "jenkins" chart repository
Update Complete. ⎈Happy Helming!⎈
PS C:\devbox> helm search repo hashicorp/vault
NAME            CHART VERSION   APP VERSION     DESCRIPTION
hashicorp/vault 0.24.0          1.13.1          Official HashiCorp Vault Chart
-->

### 2. Deploy Vault Helm Chart

**Install** the latest release of the Vault Helm chart with below command:

```dos
git clone https://github.com/briansu2004/udemy-devops-real-projects.git
cd udemy-devops-real-projects\014-VaultInjectorMinikube
# helm uninstall vault
# helm list
helm install vault hashicorp/vault -f values_windows.yaml
```

<!--
```dos
PS C:\devbox\udemy-devops-real-projects\014-VaultInjectorMinikube> helm install vault hashicorp/vault -f values.yaml
NAME: vault
LAST DEPLOYED: Sat Apr  8 20:20:55 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
Thank you for installing HashiCorp Vault!

Now that you have deployed Vault, you should look over the docs on using
Vault with Kubernetes available here:

https://www.vaultproject.io/docs/

Your release is named vault. To learn more about the release, try:

  $ helm status vault
  $ helm get manifest vault

PS C:\devbox\udemy-devops-real-projects\014-VaultInjectorMinikube> helm status vault
NAME: vault
LAST DEPLOYED: Sat Apr  8 20:20:55 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
Thank you for installing HashiCorp Vault!

Now that you have deployed Vault, you should look over the docs on using
Vault with Kubernetes available here:

https://www.vaultproject.io/docs/

Your release is named vault. To learn more about the release, try:

  $ helm status vault
  $ helm get manifest vault
```
-->

### 3. Setup Vault

a. **Initiate** vault

???

```dos
vault operator init
```

**Note:** Make a note of the output. This is the only time ever you see those **unseal keys** and **root token**. If you lose it, you won't be able to seal vault any more.

b. **Unsealing** the vault </br>
Type `vault operator unseal <unseal key>`. The unseal keys are from previous output. You will need at lease **3 keys** to unseal the vault. </br>

When the value of  `Sealed` changes to **false**, the Vault is unsealed. You should see below similar output once it is unsealed

```dos
Unseal Key (will be hidden): 
Key                     Value
---                     -----
Seal Type               shamir
Initialized             true
Sealed                  false
Total Shares            5
Threshold               3
Version                 1.12.1
Build Date              2022-10-27T12:32:05Z
Storage Type            raft
Cluster Name            vault-cluster-403fc7a0
Cluster ID              772cef22-77d2-11bb-f16b-7ef69d85ac0e
HA Enabled              true
HA Cluster              n/a
HA Mode                 standby
Active Node Address     <none>
Raft Committed Index    31
Raft Applied Index      31
```

c. Sign in to Vault with **root** user </br>
Type `vault login` and enter the `<Initial Root Token>` retrieving from previous output

```dos
/ # vault login
Token (will be hidden): 
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                  Value
---                  -----
token                hvs.KtwbjaZwYBV4BPohe6Vi48BH
token_accessor       aVZzcPF3oCCIqGLzqoxvgLLC
token_duration       ∞
token_renewable      false
token_policies       ["root"]
identity_policies    []
policies             ["root"]
```

### 4. Enable Vault KV Secrets Engine Version 2 and Create a Secret
>
> Refer to <https://developer.hashicorp.com/vault/docs/secrets/kv/kv-v2>

```dos
vault secrets enable -path=internal-app kv-v2
vault kv put internal-app/database/config username=root password=changeme
```

You can **read** the data by running this:

```dos
vault kv get internal-app/database/config
```

Then you should be able to see below output

```dos
====== Data ======
Key         Value
---         -----
password    changeme
username    root
```

### 5. Configure Kubernetes authentication

Stay on the Vault pod and configure the kuberentes authentication </br>
a. **Enable** the Kuberetes atuh in the Vault

```dos
vault auth enable kubernetes
```

b. Create a **role** for the service account which is used by the deployment

```dos
vault write auth/kubernetes/config \
    kubernetes_host="https://$KUBERNETES_PORT_443_TCP_ADDR:443"

vault policy write internal-app - <<EOF
path "internal-app/data/database/config" {
  capabilities = ["read"]
}
EOF
```

> Note: Since version 2 kv has prefixed `data/`, your secret path will be `internal-app/data/database/config`, instead of `internal-app/database/config`
c. Associate the role created above to the **service account**

```dos
 vault write auth/kubernetes/role/internal-app \
    bound_service_account_names=app-sa \
    bound_service_account_namespaces=default \
    policies=internal-app \
    ttl=24h
```

### 6. Launch an Application

Apply the `app-deployment.yaml` to deploy a deployment:

```dos
kubectl apply -f app-deployment.yaml
```

Wait for the pods are **ready**

```dos
kubectl wait pods -n default -l app=nginx --for condition=Ready --timeout=1000s
```

### 7. Update the deployment to Enable the Vault Injection

To enable the vault to inject secrets into a deployment's pods, you need to patch the  code in `patch-app-deployment.yaml` into the **annotation** section of the deployment file:

```dos
kubectl patch deployment app-deployment --patch "$(cat patch-app-deployment.yaml)"
```

Once the vault sidecar is successfully injected into the app deployment's pod, you should be able to verify its presence by inspecting the pod's configuration.

```dos
kubectl exec $(kubectl get pod|grep app-deployment|awk '{print $1}') -- cat /vault/secrets/database-config.txt
```
