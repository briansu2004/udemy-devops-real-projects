# Project 013: Develop a Java Application in K8s for Monitoring ConfigMap Modifications and Content Changes

## Project Goal

In this lab, you will learn how to develop a **Java application** that interacts with the Kubernetes API to **monitor changes** to a file that is mounted by a **ConfigMap**.

<!--
It is important to note that **the file mounted by the ConfigMap is mounted as a symbolic link** (e.g. `/config/game.properties` -> `/config/..data/game.properties`-> `/config/..2023_03_02_15_51_59.1603915861/game.properties`), so your Java code should read the **symlink** instead of the file directly.
-->

## Environments

| #  | Env  | Y/N  | Recommended   |  Comment |
|---|---|---|---|---|
| 1 | Windows only | Y | Y |   |
| 2 | Windows + Ubuntu | N | Y |   |
| 3 | Mac only | Y | Y |   |
| 4 | Mac + Ubuntu | N | Y |   |

[Windows Only](01_Y_WindowsOnly.md)

<!--
[Windows Only doesn't work](01_N_WindowsOnly.md)

[With_Windows_Ubuntu](02_Y_Windows_Ubuntu.md)

[Mac Only doesn't work](03_Y_MacOnly.md)

[With_Mac_Ubuntu](04_Y_Mac_Ubuntu.md)
-->
