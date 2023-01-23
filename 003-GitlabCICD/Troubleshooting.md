# Troubleshooting

## Issue 1: check that a DNS record exists for this domain

Error out when starting up or reconfigure the gitlab server

```
RuntimeError: letsencrypt_certificate[gitlab.mydevopsrealprojects.com] (letsencrypt::http_authorization line 6) had an error: RuntimeError: acme_certificate[staging] (letsencrypt::http_authorization line 43) had an error: RuntimeError: ruby_block[create certificate for gitlab.mydevopsrealprojects.com] (letsencrypt::http_authorization line 110) had an error: RuntimeError: [gitlab.mydevopsrealprojects.com] Validation failed, unable to request certificate, Errors: [{url: https://acme-staging-v02.api.letsencrypt.org/acme/chall-v3/4042412034/SuXaFQ, status: invalid, error: {"type"=>"urn:ietf:params:acme:error:dns", "detail"=>"DNS problem: NXDOMAIN looking up A for gitlab.mydevopsrealprojects.com - check that a DNS record exists for this domain; DNS problem: NXDOMAIN looking up AAAA for gitlab.mydevopsrealprojects.com - check that a DNS record exists for this domain", "status"=>400}} ]
```

**Solution:**
Sometimes it will occur when letsencrypt is requesting for a old/used DNS record. Just give it another 15 mins to see if it will figure it out by itself. If no, You can try to reconfigure/restart the gitlab server without any change. If it doesn't work either, you can replace your gitlab domain name in `docker-compose.yaml` with more unique name and then reconfigure/restart your gitlab server.

## Issue 2: Cannot register gitlab-runner: connection refused

When running `gitlab-register`, it shows below error:

```
ERROR: Registering runner... failed                 runner=GR1348941oqts-yxX status=couldn't execute POST against https://gitlab.mydevopsrealprojects.com/api/v4/runners: Post "https://gitlab.mydevopsrealprojects.com/api/v4/runners": dial tcp 0.0.0.0:443: connect: connection refused
```

**Solution:**
Make sure to follow step 4 to regerate a new certificate with proper info and update it in gitlab-runner host/container

## Issue 3: Cannot login docker registry: x509: certificate signed by unknown authority

Cannot `docker login` to the gitlab container registry. Below error is returned

```
$ docker login registry.gitlab.mydevopsrealprojects.com:5005
Username: root
Password: 
Error response from daemon: Get "https://registry.gitlab.mydevopsrealprojects.com:5005/v2/": x509: certificate signed by unknown authority
```

**Cause:**
You are using a self-signed certificate for your gitlab container registry instead of the certificate issued by a trusted CA. The docker daemon doesnot trust the self-signed cert which causes this error

**Solution:**
You must instruct docker to trust the self-signed certificate by copying the self-signed certificate to `/etc/docker/certs.d/<your_registry_host_name>:<your_registry_host_port>/ca.crt` on the machine where running the docker login command.

```
export YOUR_GITLAB_DOMAIN=mydevopsrealprojects.com
export YOUR_GITLAB_CONTAINER=<Gitlab Container ID>

sudo mkdir -p /etc/docker/certs.d/registry.gitlab.$YOUR_GITLAB_DOMAIN:5005
sudo docker cp $YOUR_GITLAB_CONTAINER:/etc/gitlab/ssl/ca.crt /etc/docker/certs.d/registry.gitlab.$YOUR_GITLAB_DOMAIN:5005
```

## Issue 4: x509: certificate signed by unknown authority

when running `gitlab-runner registry`, failing with below error

```
ERROR: Registering runner... failed                 runner=GR1348941oqts-yxX status=couldn't execute POST against https://gitlab.mydevopsrealprojects.com/api/v4/runners: Post "https://gitlab.mydevopsrealprojects.com/api/v4/runners": x509: certificate signed by unknown authority
```

> Refer to:
> <https://www.ibm.com/docs/en/cloud-paks/cp-management/2.2.x?topic=tcnm-logging-into-your-docker-registry-fails-x509-certificate-signed-by-unknown-authority-error>
> <https://7thzero.com/blog/private-docker-registry-x509-certificate-signed-by-unknown-authority>

## Issue 5: This job is stuck because the project doesn't have any runners online assigned to it

When running the gitlab pipeline, the job gets stuck with below error

```
This job is stuck because the project doesn't have any runners online assigned to it
```

**Cause:**
Check if any gitlab runner is online. If so, most likely the job is stuck because your unners have tags but your jobs don't.

**Solution:**
You need to enable your runner without tags. Go to your project and go to "Settings" -> "CI/CD" -> Click the online gitlab runner -> Check "Indicates whether this runner can pick jobs without tags". Then your pipeline should be able to pick this runner.

> Refer to: <https://stackoverflow.com/questions/53370840/this-job-is-stuck-because-the-project-doesnt-have-any-runners-online-assigned>

## Issue 6: New runner. Has not connected yet

**Cause:**
sometimes you might see the waring icon at the left side of runner. It is due to unverified runner been setup.

![runner-is-not-ready-yet](images/issue6-runner-is-not-ready-yet.png)

**Solution:**

```
docker exec -it $(docker ps -f name=gitlab-runner -q) bash
gitlab-runner verify --delete
```

> Refer to: <https://gitlab.com/gitlab-org/gitlab-runner/-/issues/3750>
