# Lab 014: Deploy and Use Vault As Agent Sidecar Injector

## Lab goal

In thhis lab, we will go through the process of **deploying a Vault Helm chart** in a Kubernetes cluster running on **Minikube**. Once we have the Vault instance up and running, we will create a **deployment** that utilizes **Vault as a sidecar** to **inject secrets** into the pod as a file. This approach ensures that the application running in the pod has access to the necessary secrets without compromising their security by storing them in plain text within the container.

## Environments

| #  | Env  | Y/N  | Recommended   |  Comment |
|---|---|---|---|---|
| 1 | Windows only | YN | YN |   |
| 2 | Windows + Ubuntu | Y | Y |   |
| 3 | Mac only | YN | YN |   |
| 4 | Mac + Ubuntu | Y | Y |   |

[Windows Only](01_Y_WindowsOnly.md)

<!--

[With_Windows_Ubuntu](02_YN_Windows_Ubuntu.md)

[Mac Only](03_Y_MacOnly.md)

[With_Mac_Ubuntu](04_YN_Mac_Ubuntu.md)
-->
