# Project 001: ELK Monitoring

Mac + Ubuntu (vagrant vbox)

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

### 2. Add Elasticsearch CA certificates

As the communication between Elasticsearch and metricbeat is using tls, you need to add the Elasticsearch CA into the server which is going to be monitored.

a. Copy the CA certificate from one of Elasticsearch containers

```bash
docker exec -it $(docker ps -aqf "name=001-elkmonitoring-es01-1") openssl x509 -fingerprint -sha256 -in /usr/share/elasticsearch/config/certs/ca/ca.crt
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
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
sudo apt update
sudo apt install metricbeat
sudo vim /etc/metricbeat/metricbeat.yml

```

Note: Make sure below section is updated in `metricbeat.yml`

```dos
setup.kibana:
  host: "localhost:5601"

output.elasticsearch:
  hosts: ["localhost:9200"]
  #protocol: "https"
  #username: "elastic"
  #password: "changeme"
```

->

```dos
setup.kibana:
  host: "127.0.0.1:5601"

output.elasticsearch:
  hosts: ["127.0.0.1:9200"]
  protocol: "https"
  username: "elastic"
  password: "Password!2023"
```

```dos
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

### 4. Go to Kibana. In Dashboard, select "[Metricbeat System] Host overview ECS"

a. Open your browser and go to [http://192.168.33.10:5601/](http://192.168.33.10:5601/) (if the metricbeat is installed in your local host).

Enter the username (default is elastic) / password set in `.env`.

b. Click the menu icon in the top left and go to "Dashboard"

c. Select "[Metricbeat System] Host overview ECS" and you should be able to see all metric data from your local host presented in the dashboard.

![1673320776011](image/02_Y_Windows_Ubuntu/1673320776011.png)

