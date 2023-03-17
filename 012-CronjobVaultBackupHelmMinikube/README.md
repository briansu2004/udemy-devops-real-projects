# Project 012: Backup Vault in Minio

## Project Goal

In this lab, you will deploy a helm chart with a cronjob to backup vault periodically into the Minio storage

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

[With_Windows_Ubuntu](02_Y_Windows_Ubuntu.md)

<!--
[Windows Only doesn't work](01_N_WindowsOnly.md)

[Mac Only doesn't work](03_N_MacOnly.md)

[With_Mac_Ubuntu](04_Y_Mac_Ubuntu.md)
-->
