# Lab 003: GitLab CICD Pipeline

In this lab, we will setup a GitLab CICD pipeline, which will build the source code to a docker image and push it to the container registry in GitLab and then re-deploy the docker container with the latest image in our local host.

## Lab goal

Understand how to setup/configure Gitlab as CICD pipeline. Familarize with gitlab pipeline.

## Environments

| #  | Env  | Y/N  | Recommended   |  Comment |
|---|---|---|---|---|
| 1 | Windows only | N | N |   |
| 2 | Windows + Ubuntu | Y | Y |   |
| 3 | Mac only | ? | N |   |
| 4 | Mac + Ubuntu | Y | Y |   |

<!--
[Windows Only](01_N_WindowsOnly.md)
-->

[With_Windows_Ubuntu](02_Y_Windows_Ubuntu.md)

<!--
[Mac Only](03_YN_MacOnly.md)

[With_Mac_Ubuntu](04_Y_Mac_Ubuntu.md)
-->

<!--
## My troubleshooting

### [Windows] 0.0.0.0:5005 issue

```dos
C:\CodeUdemy\udemy-devops-9projects-free\003-GitlabCICD>docker compose up -d
[+] Running 2/3
 - Container 003-gitlabcicd-web-1            Starting                                                                                                                                                                        2.2s
 - Container 003-gitlabcicd-gitlab-runner-1  Started                                                                                                                                                                         2.2s
 - Container 003-gitlabcicd-hello-world-1    Running                                                                                                                                                                         0.0s
Error response from daemon: Ports are not available: exposing port TCP 0.0.0.0:5005 -> 0.0.0.0:0: listen tcp 0.0.0.0:5005: bind: An attempt was made to access a socket in a way forbidden by its access permissions.
```

Root cause: 5005 was blocked by Windows

```dos
C:\>netsh interface ipv4 show excludedportrange protocol=tcp 

Protocol tcp Port Exclusion Ranges

Start Port    End Port
----------    --------
      1045        1144
      1145        1244
      4523        4622
      4823        4922
      4923        5022
      7098        7197
      7198        7297
     14365       14464
     14765       14864
     14865       14964
     16826       16925
     16993       17092
     50000       50059     *

* - Administered port exclusions.
```

Solution: change to a different port

in `docker-compose.xml`:

```yml
      - '5055:5005'
```

```dos
C:\CodeUdemy\udemy-devops-9projects-free\003-GitlabCICD>docker compose up -d
[+] Running 3/3
 - Container 003-gitlabcicd-gitlab-runner-1  Started                                                                                                                              12.1s 
 - Container 003-gitlabcicd-hello-world-1    Started                                                                                                                              12.4s 
 - Container 003-gitlabcicd-web-1            Started                                                                                                                              12.4s 
```

### [GitLab] Set the initial root password

```yml
    image: 'gitlab/gitlab-ce:latest'
    restart: always
    hostname: 'gitlab.mydevopsrealprojects.com'
    environment:
      GITLAB_ROOT_PASSWORD: "Password2023#"
      EXTERNAL_URL: "http://gitlab.mydevopsrealprojects.com"
      GITLAB_OMNIBUS_CONFIG: |
        gitlab_rails['initial_root_password'] = "Password2023#"
        gitlab_rails['store_initial_root_password'] = true
        gitlab_rails['display_initial_root_password'] = true
    ports:
```

### [GitLab] 422

Clear the cookie and cache, then restart Chrome!
-->
