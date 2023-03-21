# Project Name: Develop a Java Application in K8s for Monitoring ConfigMap Modifications and Content Changes

## Project Goal

In this lab, you will learn how to develop a **Java application** that interacts with the Kubernetes API to **monitor changes** to a file that is mounted by a **ConfigMap**.

It is important to note that **the file mounted by the ConfigMap is mounted as a symbolic link** (e.g. `/config/game.properties` -> `/config/..data/game.properties`-> `/config/..2023_03_02_15_51_59.1603915861/game.properties`), so your Java code should read the **symlink** instead of the file directly.

## Clean up

Run below commands to remove docker containers and volumes

```bash
sudo docker compose down -v
sudo systemctl stop metricbeat
sudo systemctl disable metricbeat
sudo apt remove metricbeat
```

## Environments

| #  | Env  | Y/N  | Recommended   |  Comment |
|---|---|---|---|---|
| 1 | Windows only | YN | YN |   |
| 2 | Windows + Ubuntu | Y | Y |   |
| 3 | Mac only | Y | Y |   |
| 4 | Mac + Ubuntu | Y | Y |   |

[Windows Only](01_Y_WindowsOnly.md)

[With_Windows_Ubuntu](02_Y_Windows_Ubuntu.md)

<!--
[Windows Only doesn't work](01_N_WindowsOnly.md)

[Mac Only doesn't work](03_N_MacOnly.md)

[With_Mac_Ubuntu](04_Y_Mac_Ubuntu.md)
-->
