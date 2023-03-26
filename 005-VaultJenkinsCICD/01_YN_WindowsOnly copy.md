# Project 005: Vault Jenkins Pipeline

## Project Goal

In this article, you will learn how to integrate Vault into Jenkins pipeline, as well as the basic usage of Hashicorp Vault.

## Steps

### 1. Initiate Vault

a. **Initializing** the Vault

```bash

docker-compose up
```

```dos
docker exec -it $(docker ps -f name=vault_1 -q) sh
export VAULT_ADDR='http://127.0.0.1:8200'
vault operator init
```

**Note:** Make a note of the output. This is the only time ever you see those unseal keys and root token. If you lose it, you won't be able to seal vault any more.

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

c. Sign in to vault with **root** user </br>
Type `vault login` and enter the `Initial Root Token` retrieving from previous output

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

### 2. Enable Vault KV Secrets Engine Version 2

> Refer to <https://developer.hashicorp.com/vault/docs/secrets/kv/kv-v2>

```dos
vault secrets enable -version=2 kv-v2

vault kv put -mount=kv-v2 devops-secret username=root password=changeme
```

You can **read** the data by running this:

```dos
vault kv get -mount=kv-v2 devops-secret
```

Then you should be able to see below output

```dos
====== Data ======
Key         Value
---         -----
password    changeme
username    root

```

> Note: Since version 2 kv has prefixed `data/`, your secret path will be `kv-v2/data/devops-secret`, instead of `kv-v2/devops-secret`

### 3. Write a Vault Policy and create a token

a. **Write** a policy

```dos
cat > policy.hcl  <<EOF
path "kv-v2/data/devops-secret/*" {
  capabilities = ["create", "update","read"]
}
EOF
vault policy write first-policy policy.hcl
vault policy list
vault policy read first-policy
```

b. **Enable approle**

```dos
vault auth enable approle
```

c. Create an **role**

```dos
vault write auth/approle/role/first-role \
    secret_id_ttl=10000m \
    token_num_uses=10 \
    token_ttl=20000m \
    token_max_ttl=30000m \
    secret_id_num_uses=40 \
    token_policies=first-policy

# Check the role id
export ROLE_ID="$(vault read -field=role_id auth/approle/role/first-role/role-id)"
echo $ROLE_ID
```

> **Note:** Please make a note as it will be needed when configuring Jenkins credential

d. Create a **secret id** via the previous role

```dos
export SECRET_ID="$(vault write -f -field=secret_id auth/approle/role/first-role/secret-id)"
echo $SECRET_ID
```

> **Note:** Please make a note as it will be needed when configuring Jenkins credential

e. Create a **token** with the role ID and secret ID

```dos
apk add jq
export VAULT_TOKEN=$(vault write auth/approle/login role_id="$ROLE_ID" secret_id="$SECRET_ID" -format=json|jq .auth.client_token)
echo $VAULT_TOKEN
VAULT_TOKEN=$(echo $VAULT_TOKEN|tr -d '"')
vault token lookup | grep policies
```

f. Write a **secret** via the new token

```dos
vault kv put -mount=kv-v2 devops-secret/team-1 username2=root2 password2=changemeagain
vault kv get -mount=kv-v2 devops-secret/team-1

```

### 4. Add the role id/secret id in Jenkins

> Refer to <https://plugins.jenkins.io/hashicorp-vault-plugin/#plugin-content-vault-app-role-credential>

Login to your Jenkins website and go to **"Manage Jenkins"** -> **"Manage Credentials"** ->  **"System"** -> **"Global credentials (unrestricted)"** -> Click **"Add Credentials"** and you should fill out the page in below selection: </br>
**Kind:** Vault App Role Credential</br>
**Scope:** Global (Jenkins,nodes,items,all child items,etc)</br>
**Role ID:** <ROLE_ID from previous step></br>
**Secret ID:** <SECRET_ID from previous step></br>
**Path:** approle</br>
**Namespace:** (Leave it blank) </br>
**ID:** (the credential id you will refer within Jenkins Pipeline. i.g. vault-app-role)</br>
**Description:** Vault: AppRole Authentication</br>

### 5. Add github credential in Jenkins

Login to your Jenkins website and go to **"Manage Jenkins"** -> **"Manage Credentials"** ->  **"System"** -> **"Global credentials (unrestricted)"** -> Click **"Add Credentials"** and you should fill out the page below below selection:</br>
**Scope:** Global (Jenkins,nodes,items,all child items,etc) </br>
**Username:** (your github username)</br>
**Password:** <your github personal access token> (please refer to [here](https://docs.github.com/en/enterprise-server@3.4/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token))</br>
**ID:** (the id which will be referred in Jenkinsfile, i.g. github-token)</br>
**Description:** Github token</br>

### 6. Create a Jenkins Pipeline

a. In the Jenkins portal, click **"New Item"** in the left navigation lane, and type the item name (i.g. first-project) and select **"Pipeline"**. Click **"OK"** to configure the pipeline.</br>
b. Go to **"Pipeline"** section and select **"Pipeline script from SCM"** in the **"Definition"** field</br>
c. Select **"Git"** in **"SCM"** field</br>
d. Add `https://github.com/chance2021/devopsdaydayup.git` in **"Repository URL"** field</br>
e. Select your github credential in **"Credentials"**</br>
f. Type `*/main` in **"Branch Specifier"** field</br>
g. Type `005-VaultJenkinsCICD/Jenkinsfile` in **"Script Path"**</br>
h. Unselect **"Lightweight checkout"**</br>
