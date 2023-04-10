# Lab 001: ELK Monitoring

In this lab, we will setup a ELK (Elasticsearch/Logstash/Kibana) stack to monitor a server.

## Lab goal

Learn how to deploy a ELK with docker-compose, as well as configuring metricbeat service to collect the system metric from a server and present them in Kibana.

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
| 1 | Windows only | N | N |   |
| 2 | Windows + Ubuntu | Y | Y |   |
| 3 | Mac only | N | N |   |
| 4 | Mac + Ubuntu | Y | Y |   |

[With_Windows_Ubuntu](02_Y_Windows_Ubuntu.md)

<!--
[Windows Only doesn't work](01_N_WindowsOnly.md)

[Mac Only doesn't work](03_N_MacOnly.md)

[With_Mac_Ubuntu](04_Y_Mac_Ubuntu.md)
-->
