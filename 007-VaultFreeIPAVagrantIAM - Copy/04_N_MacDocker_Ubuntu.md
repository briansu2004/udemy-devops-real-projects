# Project 007: Managing SSH Access with Vault

Mac (Docker) + Ubunbu

Has issues:

when `docker compose up` -

```bash
007-vaultfreeipavagrantiam-freeipa-1  | Set hostname to <ipa.devopsdaydayup.org>.
007-vaultfreeipavagrantiam-freeipa-1  | Initializing machine ID from random generator.
007-vaultfreeipavagrantiam-freeipa-1  | Failed to create /init.scope control group: Read-only file system
007-vaultfreeipavagrantiam-freeipa-1  | Failed to allocate manager object: Read-only file system
007-vaultfreeipavagrantiam-freeipa-1  | [!!!!!!] Failed to allocate manager object, freezing.
007-vaultfreeipavagrantiam-freeipa-1  | Freezing execution.
```

## Scenario

We have a running **FreeIPA** system which has two users: `devops` and `bob`.

`devops` is a system admin and should have all administrator priviliages (i.g. `sudo` group), while `bob` is just a regular user.

In our **Vagrant** VM, there are two accounts.

One is `admin`, which is `sudo` user, and another one is `app-user`, which is regular user.

The goal is that the `devops` user in FreeIPA should be able to login the Vagrant VM in `admin` account, and `bob` user in FreeIPA should login as `app-user` in Vagrant VM. The SSH certificates should only last for 3 mins.

## Prerequisites

### 1. Install Docker for Mac

### 2. Install Vagrant for Mac

### 3. Config hosts

a. In our local host (Mac), update `/etc/hosts` by adding this entry: `192.168.33.10 ipa.devopsdaydayup.org`

b. In our Vagrant VM, update `/etc/hosts` by adding this entry: `0.0.0.0 ipa.devopsdaydayup.org`

```bash
vagrant up
vagrant ssh
sudo vim /etc/hosts
```

## Steps

### 1. Docker compose

```bash
git clone https://github.com/briansu2004/udemy-devops-real-projects.git
cd udemy-devops-real-projects/007-VaultFreeIPAVagrantIAM
docker compose up
```

### 2. Initiate Vault

a. **Initializing** the Vault

```bash
docker exec -it $(docker ps -f name=vault-1 -q) sh

export VAULT_ADDR='http://127.0.0.1:8200'
vault operator init
```

**Note:** Make a note of the output. This is the only time ever We see those **unseal keys** and **root token**. If we lose it, We won't be able to seal vault any more.

<!--
```bash
/vault/data # export VAULT_ADDR='http://127.0.0.1:8200'
/vault/data # vault operator init
Unseal Key 1: eV1E4JskcC73l+WwizFV2+5O6RGMwbypx4M7N4fyPOd3
Unseal Key 2: QvWY6hD/osOymynK3brCKyAh789oTfRuBB0cvRsLLM/D
Unseal Key 3: tN/CKhfGqYvUuO30WCRfvvfbOdkpoPBp8p3R5kBvwboX
Unseal Key 4: NW4pRAES4S3yvpjIaSkhA07u8AZ0VgcN9e0Wd9lbZ4JR
Unseal Key 5: /ppmTJvQdXRhsFOPfjR5/PRZWdKJPt9ZVNf3kKIjoOt/

Initial Root Token: hvs.ka0ctyNxLXxNC0LSpyPgKRDw

Vault initialized with 5 key shares and a key threshold of 3. Please securely
distribute the key shares printed above. When the Vault is re-sealed,
restarted, or stopped, you must supply at least 3 of these keys to unseal it
before it can start servicing requests.

Vault does not store the generated root key. Without at least 3 keys to
reconstruct the root key, Vault will remain permanently sealed!

It is possible to generate new unseal keys, provided you have a quorum of
existing unseal keys shares. See "vault operator rekey" for more information.
```
-->

b. **Unsealing** the vault

Type `vault operator unseal <unseal key>`. The unseal keys are from previous output. We need at lease **3 keys** to unseal the vault.

When the value of  `Sealed` changes to **false**, the Vault is unsealed. We should see below similar output once it is unsealed

```bash
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

Type `vault login` and enter the `<Initial Root Token>` retrieving from previous output.

```bash
/ # vault login
Token (will be hidden): 
Success! We are now authenticated. The token information displayed below
is already stored in the token helper. We do NOT need to run "vault login"
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

### 2. Enable **ssh-client-signer** Engine in Vault

```bash
vault secrets enable -path=ssh-client-signer ssh

vault write ssh-client-signer/config/ca generate_signing_key=true
```

<!--
```bash
/vault/data # vault secrets enable -path=ssh-client-signer ssh
Success! Enabled the ssh secrets engine at: ssh-client-signer/
/vault/data # 
/vault/data # vault write ssh-client-signer/config/ca generate_signing_key=true
Key           Value
---           -----
public_key    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCvumPJpJHYKw269OjCVUEikrOLiOmkeRbWayWrWhK2zZXQLkfi7rCM0zEXNQGIh2gRtnhQpg9HeuwUKUYiIZ1991JMW5GYtMyBIdpAJVjp9VhWutH04Kd/50w3iqIMHTizZuQYlz+Hz97kN6wcUgpyXiOSeBCrBkNQRifnOAjeXQJOIRln/Nq1REqB8t6OzT4Pb7IHsVG5ty5AmnZz7/N3OSMCrJG11u8RABqtQoOi/wJFgd+mjoBvqf0mgZmGIwQ00PUtr/v6ZGH+R5UswzduFE6exHH6RPa4lQE6zXPaP6/6duP0ppeNOQT3OO+eCSmjTTvYfcjHJTpNTN10+VfnS7AKPfQTcge/Fm7afCL2LBAbGtItkwzhXKlT1JgEGliiOXD0DcOukbdKQcbYB+Ib/ThqbMEsfZNpOVGiIrZ8ADqC6AYfxdtDi7g0h+4bWK3b+GPZ/LJ6o4AdGrZv49Voji83mes6VOSnbKeAdwSp5biOuftKBM2CduoHZhPWNiWLG+3uRzuIDMA/kw1SFoVj1FKCarKWwSQAWf5PhM0Y3ywIUw9rGn7wDTWiK7ej0EVyYCbJ3twsXPR3edzNgy5vUle8HFtfPCvZEHpkW8y/Hrr7DKn0Xqxhvd1XbUbJ9Bv03C9qEKz66s2qmyFJeFuf810jqyZ7zC5uepgmzwsoAw==
```
-->

We should get **a SSH CA public key** in the output, which will be used later on the Vagrant VM host configurations. Make a note of this key.

### 3. Create Vault **roles** for signing client SSH keys

We are going to create two roles in Vault. One is `admin-role` and another one is `user-role`.

admin-role

```bash
vault write ssh-client-signer/roles/admin-role -<<EOH
{
 "allow_user_certificates": true,
 "allowed_users": "admin",
 "allowed_extensions": "",
 "default_extensions": [
 {
 "permit-pty": ""
 }
 ],
 "key_type": "ca",
 "default_user": "admin",
 "ttl": "3m0s"
}
EOH
```

<!--
```bash
/vault/data # vault write ssh-client-signer/roles/admin-role -<<EOH
> {
>  "allow_user_certificates": true,
>  "allowed_users": "admin",
>  "allowed_extensions": "",
>  "default_extensions": [
>  {
>  "permit-pty": ""
>  }
>  ],
>  "key_type": "ca",
>  "default_user": "admin",
>  "ttl": "3m0s"
> }
> EOH
Success! Data written to: ssh-client-signer/roles/admin-role
```
-->

user-role

```bash
vault write ssh-client-signer/roles/user-role -<<EOH
{
 "allow_user_certificates": true,
 "allowed_users": "user",
 "allowed_extensions": "",
 "default_extensions": [
 {
 "permit-pty": ""
 }
 ],
 "key_type": "ca",
 "default_user": "user",
 "ttl": "3m0s"
}
EOH
```

<!--
```bash
vault write ssh-client-signer/roles/user-role -<</vault/data # vault write ssh-client-signer/roles/user-role -<<EOH
> {
>  "allow_user_certificates": true,
>  "allowed_users": "user",
>  "allowed_extensions": "",
>  "default_extensions": [
>  {
>  "permit-pty": ""
>  }
>  ],
>  "key_type": "ca",
>  "default_user": "user",
>  "ttl": "3m0s"
> }
> EOH
Success! Data written to: ssh-client-signer/roles/user-role
```
-->

### 4. Create Vault **Policies**

We are going to create policies for cooresponding roles created above.

admin-policy

```bash
vault policy write admin-policy - << EOF
# List available SSH roles
path "ssh-client-signer/roles/*" {
 capabilities = ["list"]
}
# Allow access to SSH role
path "ssh-client-signer/sign/admin-role" {
 capabilities = ["create","update"]
}
EOF
```

<!--
```bash
/vault/data # vault policy write admin-policy - << EOF
> # List available SSH roles
> path "ssh-client-signer/roles/*" {
>  capabilities = ["list"]
> }
> # Allow access to SSH role
> path "ssh-client-signer/sign/admin-role" {
>  capabilities = ["create","update"]
> }
> EOF
Success! Uploaded policy: admin-policy
```
-->

user-policy

```bash
vault policy write user-policy - << EOF
# List available SSH roles
path "ssh-client-signer/roles/*" {
 capabilities = ["list"]
}
# Allow access to SSH role
path "ssh-client-signer/sign/user-role" {
 capabilities = ["create","update"]
}
EOF
```

<!--
```bash
/vault/data # vault policy write user-policy - << EOF
> # List available SSH roles
> path "ssh-client-signer/roles/*" {
>  capabilities = ["list"]
> }
> # Allow access to SSH role
> path "ssh-client-signer/sign/user-role" {
>  capabilities = ["create","update"]
> }
> EOF
Success! Uploaded policy: user-policy
```
-->

### 5. Enable **LDAP Engine** and configure the FreeLDAP setting in Vault

```bash
vault auth enable ldap

vault write auth/ldap/config \
    url="ldaps://ipa.devopsdaydayup.org" \
    userattr="uid" \
    userdn="cn=users,cn=accounts,dc=devopsdaydayup,dc=org" \
    groupdn="cn=groups,cn=accounts,dc=devopsdaydayup,dc=org" \
    groupfilter="" \
    binddn="uid=admin,cn=users,cn=accounts,dc=devopsdaydayup,dc=org" \
    bindpass="admin123" \ <--- This is the password for FreeIPA admin user
    insecure_tls=true \
    certificate="" \
    starttls=false \
    upndomain="" \
    discoverdn=true

# Attach the policies to the roles
vault write auth/ldap/users/devops  policies=admin-policy

vault write auth/ldap/users/bob  policies=user-policy
```

<!--
```bash
/vault/data # vault auth enable ldap
Success! Enabled ldap auth method at: ldap/
/vault/data #
/vault/data # vault write auth/ldap/config \
>     url="ldaps://ipa.devopsdaydayup.org" \
>     userattr="uid" \
>     userdn="cn=users,cn=accounts,dc=devopsdaydayup,dc=org" \
>     groupdn="cn=groups,cn=accounts,dc=devopsdaydayup,dc=org" \
>     groupfilter="" \
>     binddn="uid=admin,cn=users,cn=accounts,dc=devopsdaydayup,dc=org" \
>     bindpass="admin123" \ <--- This is the password for FreeIPA admin user
sh: can't open ---: no such file
/vault/data #     insecure_tls=true \
>     certificate="" \
>     starttls=false \
>     upndomain="" \
>     discoverdn=true
/vault/data #
/vault/data # vault write auth/ldap/users/devops  policies=admin-policy
Success! Data written to: auth/ldap/users/devops
/vault/data #
/vault/data # vault write auth/ldap/users/bob  policies=user-policy
Success! Data written to: auth/ldap/users/bob
```
-->

### 6. Configure the SSH Setting in the **Vagrant VM** Host

We need Vagrant VM for this step.

```bash
vagrant up

vagrant ssh
```

> Note: The trusted CA key was generated in previous step 2

```bash
echo 'ssh-rsa <TRUSTED CA Key>' | sudo tee /etc/ssh/trusted-CA.pem

cd /etc/ssh
sudo mkdir auth_principals/
cd auth_principals/

sudo echo 'admin' |sudo tee admin
sudo echo 'user' |sudo tee app-user

sudo tee -a /etc/ssh/ssh_config > /dev/null <<EOF
    AuthorizedPrincipalsFile /etc/ssh/auth_principals/%u
    ChallengeResponseAuthentication no
    PasswordAuthentication no
    TrustedUserCAKeys /etc/ssh/trusted-CA.pem
EOF

sudo service ssh restart
```

<!--
```bash
vagrant@vagrant:~$ ls -l /etc/ssh/
total 568
-rw-r--r-- 1 root root 535195 Dec  2  2021 moduli
-rw-r--r-- 1 root root   1603 May 29  2020 ssh_config
drwxr-xr-x 2 root root   4096 May 29  2020 ssh_config.d
-rw-r--r-- 1 root root   3323 Feb 12  2022 sshd_config
drwxr-xr-x 2 root root   4096 Dec  2  2021 sshd_config.d
-rw------- 1 root root    505 Feb 12  2022 ssh_host_ecdsa_key      
-rw-r--r-- 1 root root    174 Feb 12  2022 ssh_host_ecdsa_key.pub  
-rw------- 1 root root    399 Feb 12  2022 ssh_host_ed25519_key    
-rw-r--r-- 1 root root     94 Feb 12  2022 ssh_host_ed25519_key.pub
-rw------- 1 root root   2602 Feb 12  2022 ssh_host_rsa_key        
-rw-r--r-- 1 root root    566 Feb 12  2022 ssh_host_rsa_key.pub    
-rw-r--r-- 1 root root    342 Feb 12  2022 ssh_import_id
vagrant@vagrant:~$
vagrant@vagrant:~$ echo 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDSXZT+pLmFITOlvFDbX6/+wcp0T9ywrh8RvgVwcUvV2F2/nvGbebKvvRdpr92u//UmXVx5KK7Uhw2GGLUjB9ayQEcRY+ZySWdTJ5h5ap2yapSLxuCr4iO5nfKfawvNOCkgB0ReYBm0FH5DcpPiLd1m5mfNXDOcnKWok6GwB7JX1gPeKXqZ3ZEBkps6M97EeVY32P8kgXPAAsPnVXtQI36/9r+48TN1UHkDGFbRMS/vSoYeqdLgBVv/wtpfvbR8ACd4MWBPRFFtvp9sRo3X38fQM1TCdQmv/3+EOd85DbRS/kLD6nLZ5fJvWeCOURFz2xj6mD2wORq4qTIdoEmAVRI57Sp9Z76eRxxjjndVL3E+RUEmHkLUG/On72ltJTiuimXMBtn0L9OGIOnWdApl7XeQB7FEfgFuQEITS5KVh5QPN72rFRvul00t1YbEQbZ5Kk80Bn4gASVO7ov/0WdruMFvlLxUucyHhbGsEu/eS23QlNqwR3PMnh42mkcSUzw1EGNXs0WIAxJPWJ7QqWaBFOWMT48ODJp2R67vSqH5I8l1MZ8hPze7jXRUwUMIWLKpGQ2Wwt77d720j3gGXsdl5/kpSMh9HDmZND3SUc+tUfOWsXh0Tws+IGIAxoi8YZcieH1D2Cm1smpt9aJr4pkOLPIwEEYUMaG4israEs9vY/fn2w==' | sudo tee /etc/ssh/trusted-CA.pem
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCvumPJpJHYKw269OjCVUEikrOLiOmkeRbWayWrWhK2zZXQLkfi7rCM0zEXNQGIh2gRtnhQpg9HeuwUKUYiIZ1991JMW5GYtMyBIdpAJVjp9VhWutH04Kd/50w3iqIMHTizZuQYlz+Hz97kN6wcUgpyXiOSeBCrBkNQRifnOAjeXQJOIRln/Nq1REqB8t6OzT4Pb7IHsVG5ty5AmnZz7/N3OSMCrJG11u8RABqtQoOi/wJFgd+mjoBvqf0mgZmGIwQ00PUtr/v6ZGH+R5UswzduFE6exHH6RPa4lQE6zXPaP6/6duP0ppeNOQT3OO+eCSmjTTvYfcjHJTpNTN10+VfnS7AKPfQTcge/Fm7afCL2LBAbGtItkwzhXKlT1JgEGliiOXD0DcOukbdKQcbYB+Ib/ThqbMEsfZNpOVGiIrZ8ADqC6AYfxdtDi7g0h+4bWK3b+GPZ/LJ6o4AdGrZv49Voji83mes6VOSnbKeAdwSp5biOuftKBM2CduoHZhPWNiWLG+3uRzuIDMA/kw1SFoVj1FKCarKWwSQAWf5PhM0Y3ywIUw9rGn7wDTWiK7ej0EVyYCbJ3twsXPR3edzNgy5vUle8HFtfPCvZEHpkW8y/Hrr7DKn0Xqxhvd1XbUbJ9Bv03C9qEKz66s2qmyFJeFuf810jqyZ7zC5uepgmzwsoAw==
vagrant@vagrant:~$
vagrant@vagrant:~$ ls -l /etc/ssh/
total 572
-rw-r--r-- 1 root root 535195 Dec  2  2021 moduli    
-rw-r--r-- 1 root root   1603 May 29  2020 ssh_config
drwxr-xr-x 2 root root   4096 May 29  2020 ssh_config.d
-rw-r--r-- 1 root root   3323 Feb 12  2022 sshd_config
drwxr-xr-x 2 root root   4096 Dec  2  2021 sshd_config.d
-rw------- 1 root root    505 Feb 12  2022 ssh_host_ecdsa_key
-rw-r--r-- 1 root root    174 Feb 12  2022 ssh_host_ecdsa_key.pub
-rw------- 1 root root    399 Feb 12  2022 ssh_host_ed25519_key
-rw-r--r-- 1 root root     94 Feb 12  2022 ssh_host_ed25519_key.pub
-rw------- 1 root root   2602 Feb 12  2022 ssh_host_rsa_key
-rw-r--r-- 1 root root    566 Feb 12  2022 ssh_host_rsa_key.pub
-rw-r--r-- 1 root root    342 Feb 12  2022 ssh_import_id
-rw-r--r-- 1 root root    725 Apr  2 03:09 trusted-CA.pem
vagrant@vagrant:~$ cat /etc/ssh/trusted-CA.pem
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCvumPJpJHYKw269OjCVUEikrOLiOmkeRbWayWrWhK2zZXQLkfi7rCM0zEXNQGIh2gRtnhQpg9HeuwUKUYiIZ1991JMW5GYtMyBIdpAJVjp9VhWutH04Kd/50w3iqIMHTizZuQYlz+Hz97kN6wcUgpyXiOSeBCrBkNQRifnOAjeXQJOIRln/Nq1REqB8t6OzT4Pb7IHsVG5ty5AmnZz7/N3OSMCrJG11u8RABqtQoOi/wJFgd+mjoBvqf0mgZmGIwQ00PUtr/v6ZGH+R5UswzduFE6exHH6RPa4lQE6zXPaP6/6duP0ppeNOQT3OO+eCSmjTTvYfcjHJTpNTN10+VfnS7AKPfQTcge/Fm7afCL2LBAbGtItkwzhXKlT1JgEGliiOXD0DcOukbdKQcbYB+Ib/ThqbMEsfZNpOVGiIrZ8ADqC6AYfxdtDi7g0h+4bWK3b+GPZ/LJ6o4AdGrZv49Voji83mes6VOSnbKeAdwSp5biOuftKBM2CduoHZhPWNiWLG+3uRzuIDMA/kw1SFoVj1FKCarKWwSQAWf5PhM0Y3ywIUw9rGn7wDTWiK7ej0EVyYCbJ3twsXPR3edzNgy5vUle8HFtfPCvZEHpkW8y/Hrr7DKn0Xqxhvd1XbUbJ9Bv03C9qEKz66s2qmyFJeFuf810jqyZ7zC5uepgmzwsoAw==
vagrant@vagrant:~$
vagrant@vagrant:~$ cd /etc/ssh
vagrant@vagrant:/etc/ssh$ sudo mkdir auth_principals/
vagrant@vagrant:/etc/ssh$ cd auth_principals/
vagrant@vagrant:/etc/ssh/auth_principals$ sudo echo 'admin' |sudo tee admin
admin
vagrant@vagrant:/etc/ssh/auth_principals$
vagrant@vagrant:/etc/ssh/auth_principals$ sudo echo 'user' |sudo tee app-user
user
vagrant@vagrant:/etc/ssh$
vagrant@vagrant:/etc/ssh$ sudo tee -a /etc/ssh/ssh_config > /dev/null <<EOF
>     AuthorizedPrincipalsFile /etc/ssh/auth_principals/%u
>     ChallengeResponseAuthentication no
>     PasswordAuthentication no
>     TrustedUserCAKeys /etc/ssh/trusted-CA.pem
> EOF
vagrant@vagrant:/etc/ssh$ 
vagrant@vagrant:/etc/ssh$ tail /etc/ssh/ssh_config
#   VisualHostKey no
#   ProxyCommand ssh -q -W %h:%p gateway.example.com
#   RekeyLimit 1G 1h
    SendEnv LANG LC_*
    HashKnownHosts yes
    GSSAPIAuthentication yes
    AuthorizedPrincipalsFile /etc/ssh/auth_principals/%u
    ChallengeResponseAuthentication no
    PasswordAuthentication no
    TrustedUserCAKeys /etc/ssh/trusted-CA.pem

```
-->

### 7. Create LDAP users in **FreeIPA**

<!--
update `/etc/hosts` by adding this entry: `0.0.0.0 ipa.devopsdaydayup.org`

a. In our local host, update `/etc/hosts` by adding this entry: `0.0.0.0 ipa.devopsdaydayup.org`

a. In our local host (Mac), update `/etc/hosts` by adding this entry: `192.168.33.10 ipa.devopsdaydayup.org`

???
-->

b. Open the **browser** and go to The **FreeIPA portal** (<https://ipa.devopsdaydayup.org>). Type the username as `admin` and the password as `admin123`

**Note**: they are defined in `.env` file.

c. Click **"Add"** in **"Users"** page and enter below info:

- **User login:** devops
- **First Name:** devops
- **Last Name:** devops
- **New Password:** *(Type any password We want,i.g. admin123)*
- **Verify Password:** *(Type any password We want)*

d. Click **"Add and Add Another"** to create another user `user`:

- **User login:** bob
- **First Name:** bob
- **Last Name:** li
- **New Password:** *(Type any password We want, i.g. user123)*
- **Verify Password:** *(Type any password We want)*

Click **"Add"** to finish the creation. We should be able to see two users appearing in the **"Active users"** page.

### 8. Client Configurations to login as admin user

Now we are all set in server's end. In order to have a user to login to the Vagrant Host, the user needs to **create an SSH key pair** and then send the SSH **public key** to **Vault** to be **signed** by its SSH CA. The **signed SSH certificate** will then be used to connect to the target host.

Let's go through what that may look like for FreeIPA user `devops`, who is a system administrator.

a. In our local host (Mac?), create a SSH key pair

```bash
ssh-keygen -b 2048 -t rsa -f ~/.ssh/admin-key

> Note: Just leave it blank and press Enter

ssh-add ~/.ssh/admin-key
```

b. Login to **Vault** via **LDAP** credential by posting to vault's API

```bash
# Note: This is the password for `admin` user in the Vagrant VM

cat > payload.json<<EOF
{
  "password": "admin123"  
}
EOF

sudo apt install jq -y
VAULT_ADDRESS=0.0.0.0
VAULT_TOKEN=$(curl -s \
    --request POST \
    --data @payload.json \
    http://$VAULT_ADDRESS:8200/v1/auth/ldap/login/devops |jq .auth.client_token|tr -d '"')
> Note: We can see the token in `client_token` field

echo $VAULT_TOKEN

cat > public-key.json <<EOF
{
  "public_key": "$(cat ~/.ssh/admin-key.pub)",
  "valid_principals": "admin"
}
EOF

> Note: we can retrieve the public key by running the following command: `cat ~/.ssh/admin-key.pub`

SIGNED_KEY=$(curl \
    --header "X-Vault-Token: $VAULT_TOKEN" \
    --request POST \
    --data @public-key.json \
    http://$VAULT_ADDRESS:8200/v1/ssh-client-signer/sign/admin-role | jq .data.signed_key|tr -d '"'|tr -d '\n')
echo $SIGNED_KEY
SIGNED_KEY=${SIGNED_KEY::-2}
echo $SIGNED_KEY > admin-signed-key.pub

ssh -i admin-signed-key.pub admin@192.168.33.10

# Wait for 3 mins and try again, We will see `Permission denied` error, as the certificate has expired
```

> Note: If We are in Vault container trying to login the Vagrant VM, We can use below `vault` commands as well:

```bash
vault login -method=ldap username=devops
vault write -field=signed_key ssh-client-signer/sign/admin-role \
    public_key=@$HOME/.ssh/admin-key.pub valid_principals=admin > ~/.ssh/admin-signed-key.pub
ssh-keygen -Lf admin-signed-key.pub
ssh -i ~/.ssh/admin-signed-key.pub admin@192.168.33.10
```

We can now ssh to the Vagrant VM via the signed ssh key.

We can type `whoami` to see which user account We are logging with.

`exit` the Vagrant host and wait for 3 mins, and then We can try to login again with the same command above, We will find the permission is denied, as the SSH cert is expired.

```bash
$ ssh -i admin-signed-key.pub -o IdentitiesOnly=yes admin@192.168.33.10
admin@192.168.33.10: Permission denied (publickey).
```

### 9. Client Configurations to login as non-admin user

Now we are going to login as non-admin user. In FreeIPA, it is `bob`. And in the Vagrant VM, it is `app-user`. We will be authenticated as `bob` from FreeIPA in Vault and then create a signed ssh key to login the Vagrant VM as `app-user`.

a. In our local host (Mac?), create a SSH key pair

```bash
ssh-keygen -b 2048 -t rsa -f ~/.ssh/bob-key

# Note: Just leave it blank and press Enter

ssh-add ~/.ssh/bob-key
```

b. Login to **Vault** via **LDAP** credential by posting to vault's API

```bash
# Note: Below is the password for `user` user in the Vagrant VM
cat > payload.json<<EOF
{
  "password": "user123"
}
EOF

sudo apt install jq -y
VAULT_ADDRESS=0.0.0.0
VAULT_TOKEN=$(curl -s \
    --request POST \
    --data @payload.json \
    http://$VAULT_ADDRESS:8200/v1/auth/ldap/login/bob |jq .auth.client_token|tr -d '"')

#> Note: We can see the token in `client_token` field

echo $VAULT_TOKEN

cat > public-key.json <<EOF
{
  "public_key": "$(cat ~/.ssh/bob-key.pub)",
  "valid_principals": "user"
}
EOF

#> Note: We can retrieve the public key by running the following command: `cat ~/.ssh/bob-key.pub`

SIGNED_KEY=$(curl \
    --header "X-Vault-Token: $VAULT_TOKEN" \
    --request POST \
    --data @public-key.json \
    http://$VAULT_ADDRESS:8200/v1/ssh-client-signer/sign/user-role | jq .data.signed_key|tr -d '"'|tr -d '\n')
echo $SIGNED_KEY
SIGNED_KEY=${SIGNED_KEY::-2}
echo $SIGNED_KEY > bob-signed-key.pub

ssh -i bob-signed-key.pub -i ~/.ssh/bob-key  app-user@192.168.33.10

# Wait for 3 mins and try again, We will see `Permission denied` error, as the certificate has expired
```

> Note: If We are in Vault container trying to login the Vagrant VM, We can use below `vault` commands as well:

```bash
vault login -method=ldap username=bob
vault write -field=signed_key ssh-client-signer/sign/user-role \
    public_key=@$HOME/.ssh/user-key.pub valid_principals=user > ~/.ssh/user-signed-key.pub
ssh-keygen -Lf user-signed-key.pub
ssh -i ~/.ssh/user-signed-key.pub user@192.168.33.10
```

We can now ssh to the Vagrant VM via the signed ssh key. We can type `whoami` to see which user account We are logging with.

<!--
```bash

```
-->
