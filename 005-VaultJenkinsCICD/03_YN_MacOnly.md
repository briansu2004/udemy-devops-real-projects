# Project 005: Vault Jenkins Pipeline

## Project Goal

In this article, you will learn how to integrate Vault into Jenkins pipeline, as well as the basic usage of Hashicorp Vault.


## Prerequsite - fix the docker-compose issues

We use this file `plugins.txt` to manage install plug-ins. Unfortunately, it always forces us to use the latest versions. Hence the 1st step `docker-compose up -d` is likely to fail out.

e.g.

```dos
 => => naming to docker.io/library/005-vaultjenkinscicd-vault                                                                                                                               0.0s 
 => [005-vaultjenkinscicd-jenkins 2/4] COPY plugins.txt /usr/share/jenkins/ref/plugins.txt                                                                                                  0.3s 
 => ERROR [005-vaultjenkinscicd-jenkins 3/4] RUN jenkins-plugin-cli --plugin-file /usr/share/jenkins/ref/plugins.txt                                                                        5.1s 
------
 > [005-vaultjenkinscicd-jenkins 3/4] RUN jenkins-plugin-cli --plugin-file /usr/share/jenkins/ref/plugins.txt:
#0 5.036 Multiple plugin prerequisites not met:
#0 5.036 Plugin docker-workflow:528.v7c193a_0b_e67c (via pipeline-model-definition:2.2118.v31fd5b_9944b_5->git-client:4.0.0) depends on configuration-as-code:1569.vb_72405b_80249, but there is an older version defined on the t
an older version defined on the top level - configuration-as-code:1559.v38a_b_2e3b_6b_b_7,
#0 5.036 Plugin git:4.12.1 (via git-client:4.0.0) depends on configuration-as-code:1569.vb_72405b_80249, but there is an older version defined on the top level - configuration-as-code:1559.v38a_b_2e3b_6b_b_7_b_2e3b_6b_b_7
------
failed to solve: executor failed running [/bin/sh -c jenkins-plugin-cli --plugin-file /usr/share/jenkins/ref/plugins.txt]: exit code: 1
```

The solution is to update the plugin file `plugins.txt` manually based on the error messages and then re-run `docker-compose up -d` until it can go through.

e.g.

![1672798209848](image/README/1672798209848.png)

![1672798247625](image/README/1672798247625.png)


## Steps

### 1. Initiate Vault

a. **Initializing** the Vault

```bash
rm -rf udemy-devops-real-projects
git clone https://github.com/briansu2004/udemy-devops-real-projects.git
cd udemy-devops-real-projects/005-VaultJenkinsCICD
docker-compose up
```

```dos
docker ps -f name=005-vaultjenkinscicd-vault-1 -q

docker exec -it ed63f35f2c0e sh
export VAULT_ADDR='http://127.0.0.1:8200'
vault operator init
```

**Note:**

Make a note of the output. This is the only time ever you see those unseal keys and root token. If you lose it, you won't be able to seal vault any more.

<!--
```dos
/vault/data # export VAULT_ADDR='http://127.0.0.1:8200'
/vault/data # vault operator init
Unseal Key 1: iuY9B9Lkck5et/C8arMRApxPzmZkeXsq8frkw1dVEAu7
Unseal Key 2: gD9xvAqaAOXZ1rr4m8cWiortjEonrDPVchuIulPqmVQ/
Unseal Key 3: Mqse86SEwwP6I7bUyrh5UfswBrYMesKcUHVaR49ex/hU
Unseal Key 4: CQvfvH90DJ/n+6/LjenpErH/9+9YhuLhzwepsZgTA0Wq
Unseal Key 5: PoPE97/Mltjw+4FLmNEEJe2AbAfnqUAT5KP14Mri8Gp1

Initial Root Token: hvs.yTHKugYYuP55VgHSVubNuWFV

Vault initialized with 5 key shares and a key threshold of 3. Please securely
distribute the key shares printed above. When the Vault is re-sealed,
restarted, or stopped, you must supply at least 3 of these keys to unseal it
before it can start servicing requests.

Vault does not store the generated root key. Without at least 3 keys to
reconstruct the root key, Vault will remain permanently sealed!

It is possible to generate new unseal keys, provided you have a quorum of
existing unseal keys shares. See "vault operator rekey" for more information.
/vault/data #
```
-->

b. **Unsealing** the vault

Type `vault operator unseal <unseal key>`. The unseal keys are from previous output. You will need at lease **3 keys** to unseal the vault.

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

c. Sign in to vault with **root** user

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

Login to your Jenkins website and go to **"Manage Jenkins"** -> **"Manage Credentials"** ->  **"System"** -> **"Global credentials (unrestricted)"** -> Click **"Add Credentials"** and you should fill out the page in below selection:
**Kind:** Vault App Role Credential
**Scope:** Global (Jenkins,nodes,items,all child items,etc)
**Role ID:** <ROLE_ID from previous step>
**Secret ID:** <SECRET_ID from previous step>
**Path:** approle
**Namespace:** (Leave it blank)
**ID:** (the credential id you will refer within Jenkins Pipeline. i.g. vault-app-role)
**Description:** Vault: AppRole Authentication

### 5. Add github credential in Jenkins

Login to your Jenkins website and go to **"Manage Jenkins"** -> **"Manage Credentials"** ->  **"System"** -> **"Global credentials (unrestricted)"** -> Click **"Add Credentials"** and you should fill out the page below below selection:
**Scope:** Global (Jenkins,nodes,items,all child items,etc)
**Username:** (your github username)
**Password:** <your github personal access token> (please refer to [here](https://docs.github.com/en/enterprise-server@3.4/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token))
**ID:** (the id which will be referred in Jenkinsfile, i.g. github-token)
**Description:** Github token

### 6. Create a Jenkins Pipeline

a. In the Jenkins portal, click **"New Item"** in the left navigation lane, and type the item name (i.g. first-project) and select **"Pipeline"**. Click **"OK"** to configure the pipeline.
b. Go to **"Pipeline"** section and select **"Pipeline script from SCM"** in the **"Definition"** field
c. Select **"Git"** in **"SCM"** field
d. Add `https://github.com/chance2021/devopsdaydayup.git` in **"Repository URL"** field
e. Select your github credential in **"Credentials"**
f. Type `*/main` in **"Branch Specifier"** field
g. Type `005-VaultJenkinsCICD/Jenkinsfile` in **"Script Path"**
h. Unselect **"Lightweight checkout"**
