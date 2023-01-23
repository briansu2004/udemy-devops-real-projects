# Project 001: ELK Monitoring

Windows + Ubuntu (vagrant vbox)

## Steps

### 1. Deploy a ELK stack

Clone the github repo and run the docker compose to start up the ELK stack

```bash
sudo sysctl -w vm.max_map_count=262144

git clone https://github.com/briansu2004/udemy-devops-9projects-free.git
cd udemy-devops-9projects-free/001-ELKMonitoring

# passwords are in the .env file

docker compose up -d
```

```bash
vagrant@vagrant:~/udemy-devops-9projects-free/001-ELKMonitoring$ docker ps -a
CONTAINER ID   IMAGE                                                 COMMAND                  CREATED         STATUS                              PORTS                                                 NAMES
8806b17beaf9   docker.elastic.co/kibana/kibana:8.5.0                 "/bin/tini -- /usr/l…"   3 minutes ago   Up 10 seconds (health: starting)    0.0.0.0:5601->5601/tcp, :::5601->5601/tcp             001-elkmonitoring-kibana-1
9d5c75fac6fb   docker.elastic.co/elasticsearch/elasticsearch:8.5.0   "/bin/tini -- /usr/l…"   3 minutes ago   Up 3 minutes (healthy)              9200/tcp, 9300/tcp                                    001-elkmonitoring-es03-1
4972ff552cda   docker.elastic.co/elasticsearch/elasticsearch:8.5.0   "/bin/tini -- /usr/l…"   3 minutes ago   Up 3 minutes (healthy)              9200/tcp, 9300/tcp                                    001-elkmonitoring-es02-1
03a017124c86   docker.elastic.co/elasticsearch/elasticsearch:8.5.0   "/bin/tini -- /usr/l…"   3 minutes ago   Up 3 minutes (healthy)              0.0.0.0:9200->9200/tcp, :::9200->9200/tcp, 9300/tcp   001-elkmonitoring-es01-1
ce436d0f7131   docker.elastic.co/elasticsearch/elasticsearch:8.5.0   "/bin/tini -- /usr/l…"   3 minutes ago   Exited (0) Less than a second ago                                                         001-elkmonitoring-setup-1
```

### 2. Add Elasticsearch CA certificates

As the communication between Elasticsearch and metricbeat is using tls, you need to add the Elasticsearch CA into the server which is going to be monitored.

a. Copy the CA certificate from one of Elasticsearch containers

```bash
docker exec -it $(docker ps -aqf "name=001-elkmonitoring-es01-1") openssl x509 -fingerprint -sha256 -in /usr/share/elasticsearch/config/certs/ca/ca.crt
```

```bash
vagrant@vagrant:~/udemy-devops-9projects-free/001-ELKMonitoring$ docker exec -it $(docker ps -aqf "name=001-elkmonitoring-es01-1") openssl x509 -fingerprint -sha256 -in /usr/share/elasticsearch/config/certs/ca/ca.crt
SHA256 Fingerprint=AD:CC:B7:FB:62:36:20:54:B5:A9:28:6B:FF:0B:49:A6:89:1C:13:44:66:8A:9C:A6:93:BA:25:BB:93:C0:18:07
-----BEGIN CERTIFICATE-----
MIIDSTCCAjGgAwIBAgIUYtv3Pb722Tf8sJREOQv45/p1YeUwDQYJKoZIhvcNAQEL
BQAwNDEyMDAGA1UEAxMpRWxhc3RpYyBDZXJ0aWZpY2F0ZSBUb29sIEF1dG9nZW5l
cmF0ZWQgQ0EwHhcNMjMwMTEwMDIzMzQ3WhcNMjYwMTA5MDIzMzQ3WjA0MTIwMAYD
VQQDEylFbGFzdGljIENlcnRpZmljYXRlIFRvb2wgQXV0b2dlbmVyYXRlZCBDQTCC
ASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAMk2DQWWkxgRbmNl45IJSvqX
pMoo5WQBCZ8k7x6SUk1T+1USzAL2/VjHiCaxR/Fmd0ZJIZOFafa/d0Q+gpkrlzI5
VVZXMZglr5nAiRoXR8MTSP9ImKzNpBde9IRdYoY1166t0/xSEZlliDp7EC3We04c
jE5GBH5zazUIxSAavpLITm01ys/yyf3Xbx+9fbJj9xitY/3EVhEc6jD99SdKX3cP
O5yuR5D1GoYhesGZSAEHsTjbQ9TMXmgMM/QhxttzNvU35mdhd0beOD+miEZXG4F0
5r0NYJ4Y45aVCr2r5gutKOB+WJi4eSMrnDToJTN98/6J3BCbAqf+b7AAOYXGd2kC
AwEAAaNTMFEwHQYDVR0OBBYEFKpZ2lncXkxPXPdwLqCCWaEh/lIrMB8GA1UdIwQY
MBaAFKpZ2lncXkxPXPdwLqCCWaEh/lIrMA8GA1UdEwEB/wQFMAMBAf8wDQYJKoZI
hvcNAQELBQADggEBAFjcLUxrLq+6/tRVca/le2aRgGZZ7xVTEK77aTbO8zgh/u7k
iVIiS0I7oe29SqmZIWJERoWgQxe95BiE83YAgZHWI/GIyrOBy734W5AQBnPoo+xC
tl/x7496lO0ShFBUcEKThnc3VU1KfTRjrKwVczb+tU5ljvT/1VsMXBUIvZ1DlmzS
Sz5dAesGfK4g7w+7+shNNA0dnjpgf5j2bcTrvfXUSkz+qaSPCckzjEQuGCX80qvk
zSRhDKA4ZY1IIu6ATlAwUz94vPMf20WTGmTvySibZfz9Jla7agDgiKoZqkp4mEjw
jWN3EanTMFwLG2ER37F4Oawp5D18g4E8wsJO20M=
-----END CERTIFICATE-----
```

The output will be used in the next step.

b.  Go to the local host which you want to monitor and run below command:

```bash
sudo apt-get install -y ca-certificates
cd /usr/local/share/ca-certificates/
sudo vim elasticsearch-ca.crt
# Paste the CA certificate you copied in above step and then run below command to add it to the host
cat /usr/local/share/ca-certificates/elasticsearch-ca.crt
sudo update-ca-certificates
```

### 3. Deploy metricbeat service

Deploy a metricbeat service in the monitored server to collect the metric data.

> Note: In this example, we are monitoring the local host. For other hosts, you just need to update the ELK host IP address in the `metricbeat.yaml` to make sure the metricbeat can reach the Elasticsearch.

```bash
vagrant@vagrant:/usr/local/share/ca-certificates$ docker ps 
CONTAINER ID   IMAGE                                                 COMMAND                  CREATED          STATUS                    PORTS                                                 NAMES
8806b17beaf9   docker.elastic.co/kibana/kibana:8.5.0                 "/bin/tini -- /usr/l…"   15 minutes ago   Up 12 minutes (healthy)   0.0.0.0:5601->5601/tcp, :::5601->5601/tcp             001-elkmonitoring-kibana-1
9d5c75fac6fb   docker.elastic.co/elasticsearch/elasticsearch:8.5.0   "/bin/tini -- /usr/l…"   15 minutes ago   Up 15 minutes (healthy)   9200/tcp, 9300/tcp                                    001-elkmonitoring-es03-1
4972ff552cda   docker.elastic.co/elasticsearch/elasticsearch:8.5.0   "/bin/tini -- /usr/l…"   15 minutes ago   Up 15 minutes (healthy)   9200/tcp, 9300/tcp                                    001-elkmonitoring-es02-1
03a017124c86   docker.elastic.co/elasticsearch/elasticsearch:8.5.0   "/bin/tini -- /usr/l…"   15 minutes ago   Up 15 minutes (healthy)   0.0.0.0:9200->9200/tcp, :::9200->9200/tcp, 9300/tcp   001-elkmonitoring-es01-1
```

```bash
vagrant@vagrant:/usr/local/share/ca-certificates$ docker network inspect 83d
[
    {
        "Name": "001-elkmonitoring_default",
        "Id": "83d4033257b1741af85848b185d6a86712a7b72ed35fe18f01ed87f497aab4d0",
        "Created": "2023-01-10T02:33:37.172869639Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "03a017124c86651564665a2b30bd36fc41d06fcf78cd785fbb210651127969e6": {
                "Name": "001-elkmonitoring-es01-1",
                "EndpointID": "af0532f80f1e82833a17bdca3ed8b176aac7f0572d71e351c4b4b841f24e57be",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            },
            "4972ff552cdafdd56b15bfb2ffb45ec69bac9a42e5528e2174f033fa2ec01a91": {
                "Name": "001-elkmonitoring-es02-1",
                "EndpointID": "898a45948efb1cd63535a5021d245a1a65cf672be43b4e7db68a225677d71e08",
                "MacAddress": "02:42:ac:12:00:04",
                "IPv4Address": "172.18.0.4/16",
                "IPv6Address": ""
            },
            "8806b17beaf939bf3513e551aba9b01bfc519ce8c374236bb2cc2d6371ec8378": {
                "Name": "001-elkmonitoring-kibana-1",
                "EndpointID": "03be19a1e8a0bd44be72f5b878785ab53e1785aad62b4396b028b71dc61be9b0",
                "MacAddress": "02:42:ac:12:00:06",
                "IPv4Address": "172.18.0.6/16",
                "IPv6Address": ""
            },
            "9d5c75fac6fb6857fd116743266c684963d7fe76eb4eb3061a0cb7a8e2daddbe": {
                "Name": "001-elkmonitoring-es03-1",
                "EndpointID": "9b6de56c07a0755e9dea267c5a0fa3d2edf59bca0ff45bdc7d65af40eb86ac02",
                "MacAddress": "02:42:ac:12:00:05",
                "IPv4Address": "172.18.0.5/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {
            "com.docker.compose.network": "default",
            "com.docker.compose.project": "001-elkmonitoring",
            "com.docker.compose.version": "2.14.1"
        }
    }
]
```

```bash
vagrant@vagrant:/usr/local/share/ca-certificates$ ping 172.18.0.3
PING 172.18.0.3 (172.18.0.3) 56(84) bytes of data.
64 bytes from 172.18.0.3: icmp_seq=1 ttl=64 time=0.033 ms
64 bytes from 172.18.0.3: icmp_seq=2 ttl=64 time=0.100 ms
```

```bash
vagrant@vagrant:/usr/local/share/ca-certificates$ ping 0.0.0.0
PING 0.0.0.0 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.066 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.066 ms
```

Issue

```bash
vagrant@vagrant:~$ !63
sudo metricbeat setup -e
2023-01-10T03:06:23.370Z        INFO    instance/beat.go:697    Home path: [/usr/share/metricbeat] Config path: [/etc/metricbeat] Data path: [/var/lib/metricbeat] Logs path: [/var/log/metricbeat] Hostfs Path: [/]
2023-01-10T03:06:23.376Z        INFO    instance/beat.go:705    Beat ID: 90e76fa7-e16b-45b8-a22f-4a9360ae6992
2023-01-10T03:06:24.407Z        WARN    [add_cloud_metadata]    add_cloud_metadata/provider_aws_ec2.go:79       read token request for getting IMDSv2 token returns empty: Put "http://169.254.169.254/latest/api/token": dial tcp 169.254.169.254:80: connect: network is unreachable. No token in the metadata request will be used.
2023-01-10T03:06:24.517Z        INFO    [beat]  instance/beat.go:1051   Beat info       {"system_info": {"beat": {"path": {"config": "/etc/metricbeat", "data": "/var/lib/metricbeat", "home": "/usr/share/metricbeat", "logs": "/var/log/metricbeat"}, "type": "metricbeat", "uuid": "90e76fa7-e16b-45b8-a22f-4a9360ae6992"}}}
2023-01-10T03:06:24.526Z        INFO    [beat]  instance/beat.go:1060   Build info      {"system_info": {"build": {"commit": "692b4aac606e457bd2f5ef092d2d23c2fa950828", "libbeat": "7.17.8", "time": "2022-12-03T00:45:11.000Z", "version": "7.17.8"}}}
2023-01-10T03:06:24.526Z        INFO    [beat]  instance/beat.go:1063   Go runtime info {"system_info": {"go": {"os":"linux","arch":"amd64","max_procs":1,"version":"go1.18.5"}}}
2023-01-10T03:06:24.532Z        INFO    [beat]  instance/beat.go:1067   Host info       {"system_info": {"host": {"architecture":"x86_64","boot_time":"2023-01-10T02:02:35Z","containerized":false,"name":"vagrant","ip":["127.0.0.1/8","::1/128","10.0.2.15/24","fe80::a00:27ff:fe62:67d4/64","192.168.33.10/24","fe80::a00:27ff:fe17:421/64","10.0.0.45/24","fe80::a00:27ff:fe21:62af/64","172.17.0.1/16","172.18.0.1/16","fe80::42:2fff:fe3d:d121/64","fe80::3428:bdff:fe0b:c274/64","fe80::8080:13ff:fe24:d10c/64","fe80::a059:73ff:fe2e:1a52/64","fe80::ecab:e1ff:fee9:defc/64"],"kernel_version":"5.4.0-42-generic","mac":["08:00:27:62:67:d4","08:00:27:17:04:21","08:00:27:21:62:af","02:42:9a:fa:49:c1","02:42:2f:3d:d1:21","36:28:bd:0b:c2:74","82:80:13:24:d1:0c","a2:59:73:2e:1a:52","ee:ab:e1:e9:de:fc"],"os":{"type":"linux","family":"debian","platform":"ubuntu","name":"Ubuntu","version":"20.04.5 LTS (Focal Fossa)","major":20,"minor":4,"patch":5,"codename":"focal"},"timezone":"UTC","timezone_offset_sec":0,"id":"374d53f9027544c8afbc397dea1740a8"}}}
2023-01-10T03:06:24.534Z        INFO    [beat]  instance/beat.go:1096   Process info    {"system_info": {"process": {"capabilities": {"inheritable":null,"permitted":["chown","dac_override","dac_read_search","fowner","fsetid","kill","setgid","setuid","setpcap","linux_immutable","net_bind_service","net_broadcast","net_admin","net_raw","ipc_lock","ipc_owner","sys_module","sys_rawio","sys_chroot","sys_ptrace","sys_pacct","sys_admin","sys_boot","sys_nice","sys_resource","sys_time","sys_tty_config","mknod","lease","audit_write","audit_control","setfcap","mac_override","mac_admin","syslog","wake_alarm","block_suspend","audit_read"],"effective":["chown","dac_override","dac_read_search","fowner","fsetid","kill","setgid","setuid","setpcap","linux_immutable","net_bind_service","net_broadcast","net_admin","net_raw","ipc_lock","ipc_owner","sys_module","sys_rawio","sys_chroot","sys_ptrace","sys_pacct","sys_admin","sys_boot","sys_nice","sys_resource","sys_time","sys_tty_config","mknod","lease","audit_write","audit_control","setfcap","mac_override","mac_admin","syslog","wake_alarm","block_suspend","audit_read"],"bounding":["chown","dac_override","dac_read_search","fowner","fsetid","kill","setgid","setuid","setpcap","linux_immutable","net_bind_service","net_broadcast","net_admin","net_raw","ipc_lock","ipc_owner","sys_module","sys_rawio","sys_chroot","sys_ptrace","sys_pacct","sys_admin","sys_boot","sys_nice","sys_resource","sys_time","sys_tty_config","mknod","lease","audit_write","audit_control","setfcap","mac_override","mac_admin","syslog","wake_alarm","block_suspend","audit_read"],"ambient":null}, "cwd": "/home/vagrant", "exe": "/usr/share/metricbeat/bin/metricbeat", "name": 
"metricbeat", "pid": 59457, "ppid": 59456, "seccomp": {"mode":"disabled","no_new_privs":false}, "start_time": "2023-01-10T03:06:22.840Z"}}}
2023-01-10T03:06:24.535Z        INFO    instance/beat.go:291    Setup Beat: metricbeat; Version: 7.17.8
2023-01-10T03:06:24.536Z        INFO    [index-management]      idxmgmt/std.go:184      Set output.elasticsearch.index to 'metricbeat-7.17.8' as ILM is enabled.
2023-01-10T03:06:24.537Z        INFO    [esclientleg]   eslegclient/connection.go:105   elasticsearch url: http://172.18.0.3:9200
2023-01-10T03:06:24.539Z        INFO    [publisher]     pipeline/module.go:113  Beat name: vagrant
2023-01-10T03:06:24.611Z        INFO    [esclientleg]   eslegclient/connection.go:105   elasticsearch url: http://172.18.0.3:9200
2023-01-10T03:06:24.634Z        ERROR   [esclientleg]   eslegclient/connection.go:232   error connecting to Elasticsearch at http://172.18.0.3:9200: Get "http://172.18.0.3:9200": EOF
2023-01-10T03:06:24.638Z        ERROR   instance/beat.go:1026   Exiting: couldn't connect to any of the configured Elasticsearch hosts. Errors: [error connecting to Elasticsearch at http://172.18.0.3:9200: Get "http://172.18.0.3:9200": EOF]
Exiting: couldn't connect to any of the configured Elasticsearch hosts. Errors: [error connecting to Elasticsearch at http://172.18.0.3:9200: Get "http://172.18.0.3:9200": EOF]
vagrant@vagrant:~$
```

```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
sudo apt update
sudo apt install metricbeat
sudo vim /etc/metricbeat/metricbeat.yml

# Note: Make sure below section is updated in `metricbeat.yml`

setup.kibana:
  host: "localhost:5601"

output.elasticsearch:
  hosts: ["localhost:9200"]
  #protocol: "https"
  #username: "elastic"
  #password: "changeme"

->

setup.kibana:
  host: "127.0.0.1:5601"

output.elasticsearch:
  hosts: ["127.0.0.1:9200"]
  protocol: "https"
  username: "elastic"
  password: "Password!2023"


sudo metricbeat setup -e

# if you are in wsl use following command
#
# sudo service metricbeat start
# sudo service metricbeat status
#
# sudo service --status-all

sudo systemctl start metricbeat
sudo systemctl status metricbeat
```

```bash
vagrant@vagrant:~$ sudo vim /etc/metricbeat/metricbeat.yml
vagrant@vagrant:~$ sudo metricbeat setup -e
2023-01-10T03:14:48.286Z        INFO    instance/beat.go:697    Home path: [/usr/share/metricbeat] Config path: [/etc/metricbeat] Data path: [/var/lib/metricbeat] Logs path: [/var/log/metricbeat] Hostfs Path: [/]
2023-01-10T03:14:48.290Z        INFO    instance/beat.go:705    Beat ID: 90e76fa7-e16b-45b8-a22f-4a9360ae6992
2023-01-10T03:14:49.305Z        WARN    [add_cloud_metadata]    add_cloud_metadata/provider_aws_ec2.go:79       read token request for getting IMDSv2 token returns empty: Put "http://169.254.169.254/latest/api/token": dial tcp 169.254.169.254:80: connect: network is unreachable. No token in the metadata request will be used.
2023-01-10T03:14:49.407Z        INFO    [beat]  instance/beat.go:1051   Beat info       {"system_info": {"beat": {"path": {"config": "/etc/metricbeat", "data": "/var/lib/metricbeat", "home": "/usr/share/metricbeat", "logs": "/var/log/metricbeat"}, "type": "metricbeat", "uuid": "90e76fa7-e16b-45b8-a22f-4a9360ae6992"}}}
2023-01-10T03:14:49.420Z        INFO    [beat]  instance/beat.go:1060   Build info      {"system_info": {"build": {"commit": "692b4aac606e457bd2f5ef092d2d23c2fa950828", "libbeat": "7.17.8", "time": "2022-12-03T00:45:11.000Z", "version": "7.17.8"}}}
2023-01-10T03:14:49.420Z        INFO    [beat]  instance/beat.go:1063   Go runtime info {"system_info": {"go": {"os":"linux","arch":"amd64","max_procs":1,"version":"go1.18.5"}}}
2023-01-10T03:14:49.421Z        INFO    [beat]  instance/beat.go:1067   Host info       {"system_info": {"host": {"architecture":"x86_64","boot_time":"2023-01-10T02:02:35Z","containerized":false,"name":"vagrant","ip":["127.0.0.1/8","::1/128","10.0.2.15/24","fe80::a00:27ff:fe62:67d4/64","192.168.33.10/24","fe80::a00:27ff:fe17:421/64","10.0.0.45/24","fe80::a00:27ff:fe21:62af/64","172.17.0.1/16","172.18.0.1/16","fe80::42:2fff:fe3d:d121/64","fe80::3428:bdff:fe0b:c274/64","fe80::8080:13ff:fe24:d10c/64","fe80::a059:73ff:fe2e:1a52/64","fe80::ecab:e1ff:fee9:defc/64"],"kernel_version":"5.4.0-42-generic","mac":["08:00:27:62:67:d4","08:00:27:17:04:21","08:00:27:21:62:af","02:42:9a:fa:49:c1","02:42:2f:3d:d1:21","36:28:bd:0b:c2:74","82:80:13:24:d1:0c","a2:59:73:2e:1a:52","ee:ab:e1:e9:de:fc"],"os":{"type":"linux","family":"debian","platform":"ubuntu","name":"Ubuntu","version":"20.04.5 LTS (Focal Fossa)","major":20,"minor":4,"patch":5,"codename":"focal"},"timezone":"UTC","timezone_offset_sec":0,"id":"374d53f9027544c8afbc397dea1740a8"}}}
2023-01-10T03:14:49.427Z        INFO    [beat]  instance/beat.go:1096   Process info    {"system_info": {"process": {"capabilities": {"inheritable":null,"permitted":["chown","dac_override","dac_read_search","fowner","fsetid","kill","setgid","setuid","setpcap","linux_immutable","net_bind_service","net_broadcast","net_admin","net_raw","ipc_lock","ipc_owner","sys_module","sys_rawio","sys_chroot","sys_ptrace","sys_pacct","sys_admin","sys_boot","sys_nice","sys_resource","sys_time","sys_tty_config","mknod","lease","audit_write","audit_control","setfcap","mac_override","mac_admin","syslog","wake_alarm","block_suspend","audit_read"],"effective":["chown","dac_override","dac_read_search","fowner","fsetid","kill","setgid","setuid","setpcap","linux_immutable","net_bind_service","net_broadcast","net_admin","net_raw","ipc_lock","ipc_owner","sys_module","sys_rawio","sys_chroot","sys_ptrace","sys_pacct","sys_admin","sys_boot","sys_nice","sys_resource","sys_time","sys_tty_config","mknod","lease","audit_write","audit_control","setfcap","mac_override","mac_admin","syslog","wake_alarm","block_suspend","audit_read"],"bounding":["chown","dac_override","dac_read_search","fowner","fsetid","kill","setgid","setuid","setpcap","linux_immutable","net_bind_service","net_broadcast","net_admin","net_raw","ipc_lock","ipc_owner","sys_module","sys_rawio","sys_chroot","sys_ptrace","sys_pacct","sys_admin","sys_boot","sys_nice","sys_resource","sys_time","sys_tty_config","mknod","lease","audit_write","audit_control","setfcap","mac_override","mac_admin","syslog","wake_alarm","block_suspend","audit_read"],"ambient":null}, "cwd": "/home/vagrant", "exe": "/usr/share/metricbeat/bin/metricbeat", "name": 
"metricbeat", "pid": 63349, "ppid": 63348, "seccomp": {"mode":"disabled","no_new_privs":false}, "start_time": "2023-01-10T03:14:47.859Z"}}}
2023-01-10T03:14:49.427Z        INFO    instance/beat.go:291    Setup Beat: metricbeat; Version: 7.17.8
2023-01-10T03:14:49.427Z        INFO    [index-management]      idxmgmt/std.go:184      Set output.elasticsearch.index to 'metricbeat-7.17.8' as ILM is enabled.
2023-01-10T03:14:49.427Z        INFO    [esclientleg]   eslegclient/connection.go:105   elasticsearch url: https://127.0.0.1:9200
2023-01-10T03:14:49.432Z        INFO    [publisher]     pipeline/module.go:113  Beat name: vagrant
2023-01-10T03:14:49.472Z        INFO    [esclientleg]   eslegclient/connection.go:105   elasticsearch url: https://127.0.0.1:9200
2023-01-10T03:14:49.558Z        INFO    [esclientleg]   eslegclient/connection.go:285   Attempting to connect to Elasticsearch version 8.5.0
Overwriting ILM policy is disabled. Set `setup.ilm.overwrite: true` for enabling.

2023-01-10T03:14:49.564Z        INFO    [index-management]      idxmgmt/std.go:261      Auto ILM enable success.
2023-01-10T03:14:50.217Z        INFO    [index-management.ilm]  ilm/std.go:180  ILM policy metricbeat successfully created.
2023-01-10T03:14:50.217Z        INFO    [index-management]      idxmgmt/std.go:397      Set setup.template.name to '{metricbeat-7.17.8 {now/d}-000001}' as ILM is enabled.
2023-01-10T03:14:50.217Z        INFO    [index-management]      idxmgmt/std.go:402      Set setup.template.pattern to 'metricbeat-7.17.8-*' as ILM is enabled.
2023-01-10T03:14:50.217Z        INFO    [index-management]      idxmgmt/std.go:436      Set settings.index.lifecycle.rollover_alias in template to {metricbeat-7.17.8 {now/d}-000001} as ILM is enabled.
2023-01-10T03:14:50.217Z        INFO    [index-management]      idxmgmt/std.go:440      Set settings.index.lifecycle.name in template to {metricbeat {"policy":{"phases":{"hot":{"actions":{"rollover":{"max_age":"30d","max_size":"50gb"}}}}}}} as ILM is enabled.
2023-01-10T03:14:50.238Z        INFO    template/load.go:197    Existing template will be overwritten, as overwrite is enabled.
2023-01-10T03:14:50.360Z        INFO    [add_cloud_metadata]    add_cloud_metadata/add_cloud_metadata.go:101    add_cloud_metadata: hosting provider type not detected.
2023-01-10T03:14:52.126Z        INFO    template/load.go:131    Try loading template metricbeat-7.17.8 to Elasticsearch
2023-01-10T03:14:54.842Z        INFO    template/load.go:123    Template with name "metricbeat-7.17.8" loaded.
2023-01-10T03:14:54.843Z        INFO    [index-management]      idxmgmt/std.go:297      Loaded index template.
2023-01-10T03:15:00.944Z        INFO    [index-management.ilm]  ilm/std.go:140  Index Alias metricbeat-7.17.8 successfully created.
Index setup finished.
Loading dashboards (Kibana must be running and reachable)
2023-01-10T03:15:00.945Z        INFO    kibana/client.go:180    Kibana url: http://127.0.0.1:5601
2023-01-10T03:15:05.465Z        INFO    kibana/client.go:180    Kibana url: http://127.0.0.1:5601
```

### 4. Go to Kibana. In Dashboard, select "[Metricbeat System] Host overview ECS"

a. Open your browser and go to [http://192.168.33.10:5601/](http://192.168.33.10:5601/) (if the metricbeat is installed in your local host).

Enter the username (default is elastic) / password set in `.env`.

b. Click the menu icon in the top left and go to "Dashboard"

c. Select "[Metricbeat System] Host overview ECS" and you should be able to see all metric data from your local host presented in the dashboard.

![1673320776011](image/02_Y_Windows_Ubuntu/1673320776011.png)
