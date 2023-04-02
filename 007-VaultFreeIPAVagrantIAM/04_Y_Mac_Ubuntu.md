# Project 007: Managing SSH Access with Vault

Mac + Ubunbu (Docker)

TODO:

Change ipa.devopsdaydayup.org to ipa.mydevopsrealprojects.com

need to update `.env` file as well.

Issues:

<!--
when `docker compose`

```bash
Attaching to 007-vaultfreeipavagrantiam-freeipa-1, 007-vaultfreeipavagrantiam-vault-1
007-vaultfreeipavagrantiam-vault-1    | ==> Vault server configuration:
007-vaultfreeipavagrantiam-vault-1    | 
007-vaultfreeipavagrantiam-vault-1    |              Api Address: http://127.0.0.1:8200
007-vaultfreeipavagrantiam-vault-1    |                      Cgo: disabled
007-vaultfreeipavagrantiam-vault-1    |          Cluster Address: https://127.0.0.1:8201
007-vaultfreeipavagrantiam-vault-1    |               Go Version: go1.19.2
007-vaultfreeipavagrantiam-vault-1    |               Listener 1: tcp (addr: "0.0.0.0:8200", cluster address: "0.0.0.0:8201", max_request_duration: "1m30s", max_request_size: "33554432", tls: "disabled")
007-vaultfreeipavagrantiam-vault-1    |                Log Level: info
007-vaultfreeipavagrantiam-vault-1    |                    Mlock: supported: true, enabled: false
007-vaultfreeipavagrantiam-vault-1    |            Recovery Mode: false
007-vaultfreeipavagrantiam-vault-1    |                  Storage: raft (HA available)
007-vaultfreeipavagrantiam-vault-1    |                  Version: Vault v1.12.1, built 2022-10-27T12:32:05Z
007-vaultfreeipavagrantiam-vault-1    |              Version Sha: e34f8a14fb7a88af4640b09f3ddbb5646b946d9c
007-vaultfreeipavagrantiam-vault-1    | 
007-vaultfreeipavagrantiam-vault-1    | ==> Vault server started! Log data will stream in below:
007-vaultfreeipavagrantiam-vault-1    | 
007-vaultfreeipavagrantiam-vault-1    | 2023-04-02T13:56:03.810Z [INFO]  proxy environment: http_proxy="" https_proxy="" no_proxy=""
007-vaultfreeipavagrantiam-vault-1    | 2023-04-02T13:56:03.822Z [INFO]  core: Initializing version history cache for core
Error response from daemon: driver failed programming external connectivity on endpoint 007-vaultfreeipavagrantiam-freeipa-1 (fbf037e0e3c777469c8e41405b91ea029a8f7dacc686356ccecd80db6f51b291): Error starting userland proxy: listen tcp4 0.0.0.0:53: bind: address already in use
```
-->

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

```bash
cat /etc/hosts
ping ipa.devopsdaydayup.org
```

b. In our Vagrant VM, update `/etc/hosts` by adding this entry: `0.0.0.0 ipa.devopsdaydayup.org`

```bash
vagrant up
vagrant ssh

sudo vim /etc/hosts

cat /etc/hosts
ping ipa.devopsdaydayup.org
```

### 4. Config Vagrant VM

Run below commands to stop systemd-resolved

```bash
sudo systemctl disable systemd-resolved
sudo systemctl stop systemd-resolved
```

Add below entry to `/etc/resolv.conf`

```bash
nameserver 8.8.8.8
nameserver 8.8.4.4
```

<!--
Clean up:

```bash
docker container ls -a
docker volume ls 
docker image ls 

docker volume ls -q | xargs docker volume rm
```
-->

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

#export VAULT_ADDR='http://127.0.0.1:8200'
#export VAULT_ADDR='http://0.0.0.0:8200'
export VAULT_ADDR='http://192.168.33.10:8200'
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
public_key    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDJp1VviwBqjtXiBzti+WOlO4nmUJgBJv4nJGLCJuSdgpdQuFtrpAkiOpdxIDjrIEKVe3Gg84OcsokxvXgN9bgkA3s+QE0XdfaOQ9TQOMxWYJYVNVGky7LvmW3/TrFhOu6oaPxpgtyrPDUS3ZCrHIMxKZZcHFL1ZxloZe/So9bpUtctYlSTLM68HT6VJQGFRLBSD08cLrOJ5q8FD5IM4kI6XK5swF5HRindcoPht0nmBbU9IDo3Lkgff1uT2r+xnXB0EsP9m4PXpOthVqgNmL0uwBK3IjGvaFcKn2NDQ3r/7bdoAZ1tCsr3KNfGNzivUqJVq13Ybm3x/+VrS5waT2RDtST2fooBwtedMQeqBsQ32aXobwi9tEmNB3qOCHZMcc/vncwas7WbFuHBFNJX2KOaOPRlhGuHrVuk91GRB+cl0PFr0N1kr5mbVPZcRY5jE8ZD1CK9QCaF5mGQUqpmQzhD5AMuGahQsB7dNKWqwAc6U4b5KhUwA2tMnIu3xr4cu2WkhbFI1hQg6wytCzqq9QZdG5FUBpcnzhQ+yxXrs7zR42qh839I6xrC6Kby0c7dzfYoEigUsmWGjKHlHOo/ARIAtOrVOLvWpS2jRyI2HQDRNAv+phpcV8pt7kv+m6L5vX8t1oCd5G9aNLj1nWHfhTFuU8w4YZRP4vrh9GziAhv2CQ==
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
    bindpass="admin123" \
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
vagrant@vagrant:~$ echo 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC3+ayyUcQCxcUDWgAZ4KoD83ZGkx+T/NMyKZwu5d9Ok0rLeeQGpRgVKaEaKK4kOk5aLQzqQ4fK1z3O15IILk08ysI0oPEKyhg/QepWwRRCA6jnmU3Nbmff6pBhkD2smaUgctcwSf/bgGhOavQ3z0m5Ukb/Aw6KFRVpPFe5njiYcpnD1TyWAvKEIwd5FK3VNyczr5a7XsQnScAEZsNdY8dq3I33tWeabESu7jj/MuYyApjxWkRlVSueOQTY9OlYDF2TR2Pwv69UxXDKPmYwBt1z8d37tH4Cnb7w+K+Im1KzkXcF+HODOgyX9Fv7lAuJWTOnrg9eGK5mNcVyHQR3xJPxWtIkkyfaroVjdYbE7LjuZ/J8rbYAQVkphmu5pp+wQcUMdeX+Mgv0761mZMGg+UdnRJzqHHtbQAxRHdZYcO9TvkRoThPPlmhKDpYgFfp2pCJCuEC0taZJD393UTzNGXscrlr48mREGX1m42ye8CghFfkK/Fi4JK3ePPKxA4pz82P813Q4HsgLKNP/5wFX8pIxr7dRTmOEtS5Uwu9kta3M0y+cg7NErE0ih19VAOUjKvj3wXgukpAOcjVhLJYG3hSu5M8YXp9aOY6IPW0ArP9cZHes2C6AA9at8UHEOmlXUaj0TzFW7CyQz+vIifDlSz5IjdSk8HJH8he8NPOrjvbyUQ==' | sudo tee /etc/ssh/trusted-CA.pem
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
- **New Password:** *(e.g. admin123)*
- **Verify Password:** *(e.g. admin123)*

d. Click **"Add and Add Another"** to create another user `user`:

- **User login:** bob
- **First Name:** bob
- **Last Name:** li
- **New Password:** *(e.g. user123)*
- **Verify Password:** *(e.g. user123)*

Click **"Add"** to finish the creation.

We should be able to see two users appearing in the **"Active users"** page.

### 8. Client Configurations to login as admin user

Now we are all set in server's end. In order to have a user to login to the Vagrant Host, the user needs to **create an SSH key pair** and then send the SSH **public key** to **Vault** to be **signed** by its SSH CA. The **signed SSH certificate** will then be used to connect to the target host.

Let's go through what that may look like for FreeIPA user `devops`, who is a system administrator.

a. In our local host (Mac? Ubuntu), create a SSH key pair

```bash
ssh-keygen -b 2048 -t rsa -f ~/.ssh/admin-key
```

> Note: Just leave it blank and press Enter

```bash
ssh-add ~/.ssh/admin-key
```

If there are issues,

```bash
apk add openssh
eval `ssh-agent -s`
ssh-add
```

->

```bash
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install openssh-server
systemctl status ssh
sudo systemctl start ssh

eval `ssh-agent -s`
ssh-add
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

#VAULT_ADDRESS=0.0.0.0
VAULT_ADDRESS=192.168.33.10

VAULT_TOKEN=$(curl -s \
    --request POST \
    --data @payload.json \
    http://$VAULT_ADDRESS:8200/v1/auth/ldap/login/devops |jq .auth.client_token|tr -d '"')

# Note: We can see the token in `client_token` field

echo $VAULT_TOKEN

cat > public-key.json <<EOF
{
  "public_key": "$(cat ~/.ssh/admin-key.pub)",
  "valid_principals": "admin"
}
EOF

# Note: we can retrieve the public key by running the following command: `cat ~/.ssh/admin-key.pub`

SIGNED_KEY=$(curl \
    --header "X-Vault-Token: $VAULT_TOKEN" \
    --request POST \
    --data @public-key.json \
    http://$VAULT_ADDRESS:8200/v1/ssh-client-signer/sign/admin-role | jq .data.signed_key|tr -d '"'|tr -d '\n')

echo $SIGNED_KEY

#SIGNED_KEY=${SIGNED_KEY::-2}

echo $SIGNED_KEY > admin-signed-key.pub

ssh -i admin-signed-key.pub admin@192.168.33.10
```

Wait for 3 mins and try again, we will see `Permission denied` error, as the certificate has expired.

<!--
curl -s --request POST --data @payload.json http://0.0.0.0:8200/v1/auth/ldap/login/devops
curl -s --request POST --data @payload.json http://0.0.0.0:8200/v1/auth/ldap/login/devops | jq .auth.client_token|tr -d '"'
curl -s --request POST --data @payload.json http://127.0.0.1:8200/v1/auth/ldap/login/devops
curl -s --request POST --data @payload.json http://192.168.33.10:8200/v1/auth/ldap/login/devops

```bash
DevOps 🚀 devbox % curl -s --request POST --data @payload.json http://192.168.33.10:8200/v1/auth/ldap/login/devops
{"request_id":"9f5e4a4c-d740-3df7-e10d-b55edb6caba2","lease_id":"","renewable":false,"lease_duration":0,"data":{},"wrap_info":null,"warnings":["no LDAP groups found in groupDN \"cn=groups,cn=accounts,dc=devopsdaydayup,dc=org\"; only policies from locally-defined groups available"],"auth":{"client_token":"hvs.CAESINxyNjyt6HdvSxGFBCr1d7yHCoA6nzPRTdpUOpa_fgPWGh4KHGh2cy5US0ZZbTM4Z2tQek5WMURuOWRlYUg3TWs","accessor":"YVPFfa5z0CcNeSNC695Li2P1","policies":["admin-policy","default"],"token_policies":["admin-policy","default"],"metadata":{"username":"devops"},"lease_duration":31622400,"renewable":true,"entity_id":"e0d59e79-19ef-9161-fbd8-8ca2e7f98778","token_type":"service","orphan":true,"mfa_requirement":null,"num_uses":0}}

DevOps 🚀 devbox % curl -s --request POST --data @payload.json http://192.168.33.10:8200/v1/auth/ldap/login/devops | jq .auth.client_token|tr -d '"'
hvs.CAESIEUFe5aHjDu8o6FrRwnuqYrJwxrHxWA1pS9r_Msm9B31Gh4KHGh2cy5uV1FVN3E2bUNKckRTdVI0djBrQVJrdWc

# this command will have different results for all runs
DevOps 🚀 devbox % curl \
    --header "X-Vault-Token: $VAULT_TOKEN" \
    --request POST \
    --data @public-key.json \
    http://$VAULT_ADDRESS:8200/v1/ssh-client-signer/sign/admin-role
{"request_id":"1beed20b-f237-e88c-1166-8283cc3cd7f3","lease_id":"","renewable":false,"lease_duration":0,"data":{"serial_number":"bbad6c2989ae1adf","signed_key":"ssh-rsa-cert-v01@openssh.com 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\n"},"wrap_info":null,"warnings":null,"auth":null}

DevOps 🚀 devbox % curl \
    --header "X-Vault-Token: $VAULT_TOKEN" \
    --request POST \
    --data @public-key.json \
    http://$VAULT_ADDRESS:8200/v1/ssh-client-signer/sign/admin-role | jq .data.signed_key|tr -d '"'|tr -d '\n'
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  2784    0  2333  100   451  55407  10711 --:--:-- --:--:-- --:--:-- 89806
ssh-rsa-cert-v01@openssh.com AAAAHHNzaC1yc2EtY2VydC12MDFAb3BlbnNzaC5jb20AAAAgYY5xCBPKztS2/0ZosWJte0qTww9RMwdx1y0ofb8M+jUAAAADAQABAAABAQDbFiA1yCaj8lTh3JYbPM2VWrlaYNMoBZSw6HVd263gi6K1CDPDElX5bz8Z74IC6NNIS6vPYIAKB9MQ9BGnySHHbcMF2PN0JKxkZXtfjR170APD8iHhGRCN4q1rbtewuCjOVaxdUG376kK08smGfkLRMYDiuYnwu3MzmmQKTOp+QONdZB07b5UfDLzR070i24ZKRaBh7YJX3BJ7RZgci2EEu+pZCUI/w3QwThzAnLdS1wUXJ4p6RHRFvyJtnmfPv5VF8793R+MBgsXb2YTRjNlEQPZBdjgQCZghFM3hQV5bOAFmakAfr8AcOWIJpvW6dcmeGSNauty+xMMS/CADHMLh8PzdxphaiJ0AAAABAAAAUnZhdWx0LWxkYXAtZGV2b3BzLThkYzM0NjRmZmMyNDBjYTM0NmRiMjhjZWNlODAxNzhiM2RhYjUwOGQ5ZjYyMjA0YzU2MzViYmFhNTlkN2U2ZjcAAAAJAAAABWFkbWluAAAAAGQpnegAAAAAZCmeugAAAAAAAAASAAAACnBlcm1pdC1wdHkAAAAAAAAAAAAAAhcAAAAHc3NoLXJzYQAAAAMBAAEAAAIBAMmnVW+LAGqO1eIHO2L5Y6U7ieZQmAEm/ickYsIm5J2Cl1C4W2ukCSI6l3EgOOsgQpV7caDzg5yyiTG9eA31uCQDez5ATRd19o5D1NA4zFZglhU1UaTLsu+Zbf9OsWE67qho/GmC3Ks8NRLdkKscgzEpllwcUvVnGWhl79Kj1ulS1y1iVJMszrwdPpUlAYVEsFIPTxwus4nmrwUPkgziQjpcrmzAXkdGKd1yg+G3SeYFtT0gOjcuSB9/W5Pav7GdcHQSw/2bg9ek62FWqA2YvS7AErciMa9oVwqfY0NDev/tt2gBnW0Kyvco18Y3OK9SolWrXdhubfH/5WtLnBpPZEO1JPZ+igHC150xB6oGxDfZpehvCL20SY0Heo4Idkxxz++dzBqztZsW4cEU0lfYo5o49GWEa4etW6T3UZEH5yXQ8WvQ3WSvmZtU9lxFjmMTxkPUIr1AJoXmYZBSqmZDOEPkAy4ZqFCwHt00parABzpThvkqFTADa0yci7fGvhy7ZaSFsUjWFCDrDK0LOqr1Bl0bkVQGlyfOFD7LFeuzvNHjaqHzf0jrGsLopvLRzt3N9igSKBSyZYaMoeUc6j8BEgC06tU4u9alLaNHIjYdANE0C/6mGlxXym3uS/6bovm9fy3WgJ3kb1o0uPWdYd+FMW5TzDhhlE/i+uH0bOICG/YJAAACFAAAAAxyc2Etc2hhMi0yNTYAAAIAt57/aGS6WMPexNjUoConwiEB5lXmP2mdhB03LpM3T57SLWmWH0mW4EHGwq72TyqWVEkWGd+dph9yWyYuQ7Tuaea3IJPUbewBigQ8jHXC7825cVF7Hsq1dc5JGmdCfJbaFIxxAoYeIfzA+cSJlfBZ0WiJWzFgWfEGwDhxsmc3kW70UyzAgjob87rMap8h9g/tJCoCh3MPy1/XQ7UAj1J5CWnQied9AVcsNxPa0tsOCxe4jxoawvz9c2pFsGcSZGfOgsxH9PMBreGDYUxPqcC/ujzvAEdXUiTQNSVx59GCM03KeIEWHf7poSqurwA4WAKGFzpA4PG6pz1o8BKdvJxgBKSluz6cQeTEF7hlj7l6xWXXLHAyir6RFZM3aPR+x1RLIFQjym3RTq0odJbcQGhuVP1d7PJfg8avIzOcef/pbcAxa5xSIQgIyiJ6hBu130Zmw2+GWzXXrCIZCmdDq+InpncfJrWHAQWv6zDjQKd+WNs/lIn4TMkFyV7DRDfoAN/pRFwhAFrPOh0UY9JL3ivhsNAeDdu/CExEMpdhBH8awLUBV+i3xwkLi6BxIWWhtuhbKG376LOwe3JqN+2+96+PDrfk1xUFYrVaIrV+ntoB9ZfHJ8HW2AIsnXk1a9aSiqzngvN2nuZLodEMj11nLjXr2S22PydyZ4ywAEb3tCp/W6s=\n%   

DevOps 🚀 devbox % ssh -i admin-signed-key.pub admin@192.168.33.10
admin@192.168.33.10's password: 
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.4.0-42-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
New release '22.04.2 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.
```
-->

<!--
sudo apt install net-tools

vagrant@vagrant:~$ netstat -an | grep 389
tcp        0      0 0.0.0.0:7389            0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:389             0.0.0.0:*               LISTEN     
tcp        0      0 10.0.2.15:38940         91.189.91.39:80         TIME_WAIT  
tcp6       0      0 :::38927                :::*                    LISTEN     
tcp6       0      0 :::7389                 :::*                    LISTEN     
tcp6       0      0 :::389                  :::*                    LISTEN     
unix  3      [ ]         STREAM     CONNECTED     20389  
-->

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
