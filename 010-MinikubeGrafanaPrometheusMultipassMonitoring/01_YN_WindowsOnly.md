# Lab 010: Deploy Prometheus/Grafana on Minikube and Monitor The Health of Containers and VMs

Windows only

## Prerequisites

### 1. Install Docker

### 2. Install Minikube

### 3. Start Minikube

```dos
minikube start --driver=docker --kubernetes-version=v1.26.1
```

### 4. Install kubectl

### 5. Install Helm

<https://helm.sh/docs/intro/install/>

## Steps

### (Part 1) Monitoring Kuberentes nodes and containers

#### 1. Enable Minikube Dashboard

```dos
minikube dashboard
```

A Kuberentes Dashboard will pop out in our browser immediately. You can explore all Minikube resources in this UI website.

#### 2. Deploy Metrics Server

In order to collect more metrics from the cluster, we should install **metrics server** on the cluster first. You can download the manifest file as follows:

```dos
wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Then we need to **update the yaml file** by **adding** below section to **turn off the TLS verification** (more detail please see the [**Issue 1**](#issue1) in the **troubleshooting section** below)

```dos
apiVersion: apps/v1
kind: Deployment
metadata:
...
  template:
    metadata:
      labels:
        k8s-app: metrics-server
    spec:
      containers:
      - args:
        - ...
        - --kubelet-insecure-tls
        - --kubelet-preferred-address-types=InternalIP
...
```

```yml
      - args:
        - --cert-dir=/tmp
        - --secure-port=4443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
```

->

```yml
      - args:
        - --cert-dir=/tmp
        - --secure-port=4443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        - --kubelet-insecure-tls
        - --kubelet-preferred-address-types=InternalIP
```

Lastly, **apply** the manifest:

```dos
kubectl -n kube-system apply -f components.yaml
```

Once the Pod is ready, we can run below command to test out if the metric server is working.

```dos
kubectl top node
```

You should be able to see below result if it is working fine

```dos
$ kubectl top nodes
NAME       CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
minikube   622m         7%     2411Mi          15%  
```

#### 3. Add Helm Repo

Once Helm is set up properly, **add** the **repo** as follows:

```dos
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm search repo prometheus-community
```

#### 4. Deploy Prometheus Helm Chart

Install Prometheus Helm Chart by running below command:

```dos
helm install prometheus-grafana prometheus-community/kube-prometheus-stack -f values.yaml
```

#### 5. Configure Grafana Dashboard Manually

Once the deployment is settle, we can **port-forward** to the Grafana service to access the portal from our local:

```dos
kubectl -n default port-forward svc/prometheus-grafana 8888:80
```

Open our **brower** and then type the URL: [http://localhost:8888](http://localhost:8888). You should see the **Grafana login portal**. You can retrieve the **admin password** by running below command in another terminal:

```dos
kubectl get secret prometheus-grafana -o=jsonpath="{.data.admin-password}"|base64 -d
```

Enter the username (**admin**) and the password we got above (e.g. **prom-operator**), then we should be logged in the Grafana welcome board.

Go to the **Dashboards** section in the left navigation lane, and click **+New dashboard** to open a new dashboard. Follow below steps to add some **variables** before creating a panel.

a. Click **Dashboard settings**(the gear icon) in the top right

b. Go to **Variables** section

c. Click **Add variable** and add below variables

```dos
**Node**

---

- **Select variable type**: Query
- **Name**: Node
- Label: <leave it blank>
- Description: Cluster Nodes
- Show on dashboard: Label and value (Default)
- **Data source**: Prometheus
- **Query**: label_values(kubernetes_io_hostname)
- Regex: <leave it blank>
- Sort: Disabled (Default)
- Refresh: On dashboard load (Default)
- Multi-value: Unselect (Default)
- **Include All option**: Selected
- **Custom all value**: .\*

Click **Apply** to save the change

**Container**

---

- **Select variable type**: Query
- **Name**: Container
- Label: <leave it blank>
- Description: Containers in the Cluster
- Show on dashboard: Label and value (Default)
- **Data source**: Prometheus
- **Query**: label_values(container)
- Regex: <leave it blank>
- Sort: Disabled (Default)
- Refresh: On dashboard load (Default)
- Multi-value: Unselect (Default)
- **Include All option**: Selected
- **Custom all value**: .\*

Click **Apply** to save the change

**Namespace**

---

- **Select variable type**: Query
- **Name**: Namespace
- Label: <leave it blank>
- Description: Namespace in the Cluster
- Show on dashboard: Label and value (Default)
- **Data source**: Prometheus
- **Query**: label_values(namespace)
- Regex: <leave it blank>
- Sort: Disabled (Default)
- Refresh: On dashboard load (Default)
- Multi-value: Unselect (Default)
- **Include All option**: Selected
- **Custom all value**: .\*

Click **Apply** to save the change

**interval**

---

- **Select variable type**: Interval
- **Name**: interval
- Label: <leave it blank>
- Description: Interval
- **Data source**: Prometheus
- **Values**: 1m,10m,30m,1h,6h,12h,1d,7d,14d,30d
- Auto Option: Enable
- Step count: 1
- Min interval: 2m

Click **Apply** to save the change
```

Click **Save** to save the dashboard. Name it as **Container Health Status**

Once we go back to the Dashboard, we will see the **Node**/**Container**/**Namespace**/**Interval** sections are available in the top left with dropdown menu for choosing.

Now we are going to add a new **Panel**. Click **Add Panel** in the top right and click **Add a new panel** area. In the section **A**, type below query in the **Enter a PromQL query** field:

```dos
sum(kube_pod_status_phase{pod=~"^$Container.*",namespace=~"default"}) by (phase)
```

and click **Run queries** to execute the query. Make sure to choose **All** in top **Container** dropdown menu. You should see a line chart in above display area.

In order to make the graph more readable, we can change the type of charts. Just expanding the **Time series** section in the top right and search for **bar gauge** to apply.

Before saving the change, go to **Panel options** section in the right lane and type the name in **Title** field, for example, **Pod Status in Default Namespace**. And click **Apply** to save the change.

Next, we will create a **panel** to monitor the **top 5 memory intense Pods**. Again, click **Add panel** and then choose **Add a new panel**. Copy and paste below query in the query field:

```dos
topk(5,avg(container_memory_usage_bytes{}) by (pod) /1024/1024/1024)
```

and click **Run queries** to execute the query.

Expanding the **Time series** section in the top right and search for **Bar gauge** to apply. You can also change the layout orientation in **Bar gauge** -> **Orientation** section.

![Top 5 Memory Intense Pods](images/top-5-memory-intense-pod.png)

#### 6. Configure Dashboard by Importing Json file

Instead of manually configuring the dashboard, we can also **import the pre-defined dashboard from a json file**.

In the Grafana Home page, go to **Dashboards** and click **Import**. Click **Upload JSON file** and choose **pod-health-status.json** under `devopsdaydayup/010-MinikubeGrafanaPrometheusMonitoring` folder. Then we should see the dashboard imported.

#### 7. Download Dashboard Template

A variety of dashboard templates are available to meet different needs in [**Grafana Labs**](https://grafana.com/grafana/dashboards/). we can go to there and search for any dashboard we like, and just need to copy the **ID** and paste to **Grafana website** -> **Dashboard** -> **Import** -> **Import via grafana.com** and click **Load** to load the template from the website.

![Template ID](images/template-id.png)

![Template Import](images/template-import.png)

#### 8. Find Help from Your AI Friend

You can also take advanage of our AI friend (e.g. [ChatGPT](https://chat.openai.com/chat)) to generate a query as needed.

![chatgpg](images/chatgpg.png)

### (Part 2) Monitoring VMs

You can use Prometheus to monitor VMs outside of the Kubernetes cluster in addition to containers. Below are the steps to do so.

#### 1. [Option] Deploy a test VM

If we don't want to install **node exporter** in our local machine directly, we can just spin up a fresh new VM by [multipass](https://multipass.run/install).

If we are using Ubunut, we can run below commands to create a new VM via `multipass`:

```dos
snap install multipass
multipass launch
```

After deploying the VM, access it to obtain its IP address. This IP address will later be configured on the Prometheus server, allowing it to collect metrics from the VM.

```dos
multipass list
multipass shell <VM Name>
sudo apt update
sudo apt install net-tools -y
ifconfig
exit
```

You can update the **IP address** into `values.prometheus-only.yaml` file under below section:

```dos
      - job_name: multipass-vm
        static_configs:
          - targets:
            - 10.53.115.53:9100       <------Update this IP address
```

#### 2. Install Node Exporter

To monitor VMs with Prometheus, we must install the [node exporter](https://github.com/prometheus/node_exporter/releases) on each VM we want to monitor.

```dos
wget https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz
tar xvfz node_exporter-1.5.0.linux-amd64.tar.g
cd node_exporter-*.*-amd64
./node_exporter
```

#### 3. Deploy Prometheus Helm Chart

To monitor the VM, which is not part of the K8s cluster, we can deploy [another Prometheus Helm Chart](https://github.com/prometheus-community/helm-charts/tree/main/charts/prometheus) in the existing Minikube cluster. However, we may need to uninstall the previous deployment to avoid port conflict issues, if the default port has not been changed.

```dos
helm -n default uninstall prometheus-grafana
```

Then we can deploy the new Helm Chart:

```dos
helm install prometheus prometheus-community/prometheus -f values.prometheus-only.yaml
```

Once the Pod is up and running, we can **port forward** the **prometheus-server** to our local port to be access

```dos
kubectl -n default port-forward prometheus-server 9080:80
```

Open our browser and go to [http://localhost:9080](http://localhost:9080). Type below query in **Expression** field and click **Execute** to run the query:

```dos
(1 - (node_memory_MemAvailable_bytes{job="multipass-vm"} / (node_memory_MemTotal_bytes{job="multipass-vm"})))* 100
```

To view the memory usage history of the multipass VM, click the **Graph** tab and a graph will appear.
![prometheus-expression](images/prometheus-expression.png)

#### 4. Deploy Grafana Helm Chart

To enhance the visual appeal of our metrics, we will implement a Grafana for displaying them.

```dos
helm repo add grafana https://grafana.github.io/helm-charts
helm install grafana grafana/grafana 
```

Get our `admin` user password by running:

```dos
kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

The Grafana server can be **accessed via port 80** on the following DNS name from within our cluster:

```dos
export POD_NAME=$(kubectl get pods --namespace monitoring -l "app.kubernetes.io/name=grafana,app.kubernetes.io/instance=grafana" -o jsonpath="{.items[0].metadata.name}")
kubectl --namespace monitoring port-forward $POD_NAME 3000
```

**Login** with the password from above and the username: `admin` in [http://localhost:3000](http://localhost:3000) via our browser. Go to **Configuration** (gear icon in the left lane) -> **Data sources** -> Click **Add data source** -> Choose **Prometheus**. In the URL, enter `http://prometheus-server` and then **Save & test** the change.
Go to **Explorer** and type below query in the PromQL query field

```dos
(1 - (node_memory_MemAvailable_bytes{job="multipass-vm"} / (node_memory_MemTotal_bytes{job="multipass-vm"})))* 100

```

The graph will appear as shown below.

![grafana-query](images/grafana-query.png)

One useful node expertor dashboard template is available in [this website](https://grafana.com/grafana/dashboards/15172-node-exporter-for-prometheus-dashboard-based-on-11074/) or in `vm-health-status.json` file under the same folder as this README.

### (Part 3) Alert Manager

The next crucial step in setting up the monitoring system is to properly configure **alerts**.

The alert configuration will be handled by the **Alert Manager service**, which will then forward the alerts to various messaging platforms, including but not limited to Slack, Telegram, Discord, and Microsoft Teams.

In our laboratory, we will utilize **Slack** as the messaging platform. Participants can either create their own Slack channel (see [here](https://api.slack.com/messaging/webhooks) how to create a Slack webhook) or contact me at **chance.chen21@gmail.com** to join the existing one.

### 1. Update Configuration

We need to update Prometheus/Alert Manager with **Slack info**, as well as **Alert rules**. Here is the example that we have added in `values.prometheus-only.yaml` file:

```yml
serverFiles:
...
  # The following are the alert rules for specific conditions
  alerting_rules.yml:
    groups:
    - name: Test Instances
      rules:
      - alert: VM_High_Memory_Usage
        expr: 100 * (node_memory_MemTotal_bytes{job="multipass-vm"} - node_memory_MemAvailable_bytes{job="multipass-vm"}) / node_memory_MemTotal_bytes{job="multipass-vm"} > 60
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "High memory usage on VM {{ $labels.instance }}"
          description: "Memory usage on VM {{ $labels.instance }} is at {{ $value }}%, which exceeds the threshold of 60%."
      - alert: InstanceDown
        expr: up == 0
        for: 1m
        labels:
          severity: page
        annotations:
          description: '{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 5 minutes.'
          summary: 'Instance {{ $labels.instance }} down'
# The following is the receiver configuration. In the lab, we are using Slack.
alertmanager:
  config:
    global:
      resolve_timeout: 1m
      slack_api_url: 'https://hooks.slack.com/services/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'

    templates:
    - '/etc/alertmanager/*.tmpl'

    receivers:
    - name: '#slack-notification'
      slack_configs:
      - channel: 'alertmanager-testing'
        send_resolved: true

    route:
      group_wait: 10s
      group_interval: 1m
      receiver: '#slack-notification'
      repeat_interval: 1h
```

#### 2. Re-deploy Prometheus

Then, we can update the Prometheus Helm Deployment by running following command:

```dos
helm upgrade prometheus prometheus-community/prometheus -f values.prometheus-only.yaml
```

#### 3. Test

After completing the deployment, use the **Prometheus UI** to verify that the alert rule is in place. To access the UI, **port forward** the Prometheus service.

```dos
kubectl port-forward svc/prometheus-server 8888:80
```

Go to **Alerts** tab in the top and we should see 2 **Inactive** alerts there
![alert-rules-1](images/alert-rules-1.png)
To trigger the alert, navigate to the VM being monitored and run the command below to increase the RAM usage to 95%.

```dos
sudo apt update
sudo apt install stress-ng -y
stress-ng --vm 1 --vm-bytes 95% --vm-method all --verify -t 10m -v
```

Please wait a few minutes and we will receive a notification in the designated Slack channel.
![slack](images/slack.png)
