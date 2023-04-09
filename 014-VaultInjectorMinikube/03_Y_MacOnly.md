# Project 014: Deploy and Use Vault As Agent Sidecar Injector

Mac only

## Project Goal

In thhis lab, we will go through the process of **deploying a Vault Helm chart** in a Kubernetes cluster running on **Minikube**. Once we have the Vault instance up and running, we will create a **deployment** that utilizes **Vault as a sidecar** to **inject secrets** into the pod as a file. This approach ensures that the application running in the pod has access to the necessary secrets without compromising their security by storing them in plain text within the container.

## Prerequisites

### 1. Install Docker for Mac

### 2. Install Minikube for Mac

```dos
minikube start --driver=docker --kubernetes-version=v1.26.3
```

### 3. Install Helm for Mac

## Steps

### 1. Add Helm Repo

To use the Helm chart, **add the Hashicorp helm repository** and check that we have access to the chart:

```dos
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update
helm search repo hashicorp/vault
```

<!--
```bash

```
-->

### 2. Deploy Vault Helm Chart

**Install** the latest release of the Vault Helm chart with below command:

```dos
git clone https://github.com/briansu2004/udemy-devops-real-projects.git
cd udemy-devops-real-projects/014-VaultInjectorMinikube
helm install vault hashicorp/vault -f values_mac.yaml
```

<!--
```bash

```
-->

### 3. Setup Vault

a. **Initiate** vault

```bash
kubectl get pod

kubectl exec -it vault-0 -- sh

vault operator init
```

<!--
```bash
/ $ vault operator init
Unseal Key 1: FQwAnZCUIN80zM03SfndDEIxVm8/owBOgjIx9oS0fPL5
Unseal Key 2: 5diVo848wP0I43IVoFbAIuuO909Oej7wVfasTcs2NkRk
Unseal Key 3: jY3PSPd12pvpfxflj8YHBNVRjK8q+jDzzeP5G184G8wd
Unseal Key 4: zc4VY9seZw/0sg0AnnuJc4oUd9H/bST2NEFTxW1biUCZ
Unseal Key 5: Zzeclhs9hLd2uDpYYYtrSbYdSSqhkEmw8JJ5eMMER0Nt

Initial Root Token: hvs.2qvdZJViuTnS44a1Jcj8oaBq

Vault initialized with 5 key shares and a key threshold of 3. Please securely
distribute the key shares printed above. When the Vault is re-sealed,
restarted, or stopped, we must supply at least 3 of these keys to unseal it
before it can start servicing requests.

Vault does not store the generated root key. Without at least 3 keys to
reconstruct the root key, Vault will remain permanently sealed!

It is possible to generate new unseal keys, provided we have a quorum of
existing unseal keys shares. See "vault operator rekey" for more information.
```
--->

**Note:** 

Make a note of the output. This is the only time ever we see those **unseal keys** and **root token**. If we lose it, we won't be able to seal vault any more.

b. **Unsealing** the vault

Type `vault operator unseal <unseal key>`. The unseal keys are from previous output. we will need at lease **3 keys** to unseal the vault.

When the value of  `Sealed` changes to **false**, the Vault is unsealed. we should see below similar output once it is unsealed

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

c. Sign in to Vault with **root** user

Type `vault login` and enter the `<Initial Root Token>` retrieving from previous output

```dos
/ # vault login
Token (will be hidden): 
Success! we are now authenticated. The token information displayed below
is already stored in the token helper. we do NOT need to run "vault login"
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

<!--
> Refer to <https://developer.hashicorp.com/vault/docs/secrets/kv/kv-v2>
-->

```dos
vault secrets enable -path=internal-app kv-v2

vault kv put internal-app/database/config username=root password=changeme
```

<!--
```bash
/ $ vault secrets enable -path=internal-app kv-v2
Success! Enabled the kv-v2 secrets engine at: internal-app/
/ $ 
/ $ vault kv put internal-app/database/config username=root password=changeme
========== Secret Path ==========
internal-app/data/database/config

======= Metadata =======
Key                Value
---                -----
created_time       2023-04-09T21:02:11.54113053Z
custom_metadata    <nil>
deletion_time      n/a
destroyed          false
version            1
```
-->

We can **read** the data by running this:

```dos
vault kv get internal-app/database/config
```

Then we should be able to see below output:

```dos
====== Data ======
Key         Value
---         -----
password    changeme
username    root
```

<!--
```bash
/ $ vault kv get internal-app/database/config
========== Secret Path ==========
internal-app/data/database/config

======= Metadata =======
Key                Value
---                -----
created_time       2023-04-09T21:02:11.54113053Z
custom_metadata    <nil>
deletion_time      n/a
destroyed          false
version            1

====== Data ======
Key         Value
---         -----
password    changeme
username    root
```
-->

### 5. Configure Kubernetes authentication

Stay on the Vault pod and configure the kuberentes authentication

a. **Enable** the Kuberetes atuh in the Vault

```dos
vault auth enable kubernetes
```

<!--
```bash
/ $ vault auth enable kubernetes
Success! Enabled kubernetes auth method at: kubernetes/
```
-->

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

<!--
```bash
/ $ echo $KUBERNETES_PORT_443_TCP_ADDR
10.96.0.1
/ $ 
/ $ vault write auth/kubernetes/config \
>     kubernetes_host="https://$KUBERNETES_PORT_443_TCP_ADDR:443"
Success! Data written to: auth/kubernetes/config
/ $ 
/ $ vault policy write internal-app - <<EOF
> path "internal-app/data/database/config" {
>   capabilities = ["read"]
> }
> EOF
Success! Uploaded policy: internal-app
```
-->

<!--
> Note: Since version 2 kv has prefixed `data/`, your secret path will be `internal-app/data/database/config`, instead of `internal-app/database/config`
-->

c. Associate the role created above to the **service account**

```dos
 vault write auth/kubernetes/role/internal-app \
    bound_service_account_names=app-sa \
    bound_service_account_namespaces=default \
    policies=internal-app \
    ttl=24h
```

<!--
```bash
/ $  vault write auth/kubernetes/role/internal-app \
>     bound_service_account_names=app-sa \
>     bound_service_account_namespaces=default \
>     policies=internal-app \
>     ttl=24h
Success! Data written to: auth/kubernetes/role/internal-app
```
-->

### 6. Launch an Application

Apply the `app-deployment.yaml` to deploy a deployment:

```dos
kubectl apply -f app-deployment.yaml
```

<!--
```bash
devops@Brians-MacBook-Pro 014-VaultInjectorMinikube % kubectl apply -f app-deployment.yaml
deployment.apps/app-deployment created
serviceaccount/app-sa created
```
-->

Wait for the pods are **ready**

```dos
kubectl wait pods -n default -l app=nginx --for condition=Ready --timeout=1000s
```

<!--
```bash
devops@Brians-MacBook-Pro 014-VaultInjectorMinikube % kubectl wait pods -n default -l app=nginx --for condition=Ready --timeout=1000s
pod/app-deployment-d5f84c98d-9w5hz condition met
pod/app-deployment-d5f84c98d-gjtwf condition met
pod/app-deployment-d5f84c98d-kvw62 condition met
```
-->

### 7. Update the deployment to Enable the Vault Injection

To enable the vault to inject secrets into a deployment's pods, we need to patch the code in `patch-app-deployment.yaml` into the **annotation** section of the deployment file:

```dos
kubectl patch deployment app-deployment --patch "$(cat patch-app-deployment.yaml)"
```

<!--
```bash
devops@Brians-MacBook-Pro 014-VaultInjectorMinikube % kubectl patch deployment app-deployment --patch "$(cat patch-app-deployment.yaml)"
deployment.apps/app-deployment patched
```
-->

Once the vault sidecar is successfully injected into the app deployment's pod, we should be able to verify its presence by inspecting the pod's configuration.

```dos
kubectl get pod

kubectl exec $(kubectl get pod|grep app-deployment|awk '{print $1}') -- cat /vault/secrets/database-config.txt
```

<!--
```bash
kubectl exec $(kubectl get pod|grep app-deployment|awk '{print $1}') -- cat /vault/secrets/database-config.txt
Defaulted container "nginx" out of: nginx, vault-agent, vault-agent-init (init)

        export password=changeme

        export username=root
    %
```>
