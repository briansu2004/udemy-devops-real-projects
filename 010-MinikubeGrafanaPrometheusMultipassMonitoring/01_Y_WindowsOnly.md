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

<!--
### (Part 1) Monitoring Kuberentes nodes and containers
-->

#### 1. Enable Minikube dashboard

```dos
minikube dashboard=
```

A Kuberentes Dashboard will pop out in our browser immediately. We can explore all Minikube resources in this UI website.

#### 2. Deploy metrics server

In order to collect more metrics from the cluster, we should install **metrics server** on the cluster first.

<!--
We will use `components.yaml` to do it.

Download the manifest file as follows:

<https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml>

Then we need to **update the yaml file** by **adding** below section to **turn off the TLS verification**.

(more detail please see the [**Issue 1**](#issue1) in the **troubleshooting section** below)

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
        # - --kubelet-insecure-tls
        # - --kubelet-preferred-address-types=InternalIP
```

==>

```yml
        - --kubelet-insecure-tls
        - --kubelet-preferred-address-types=InternalIP
```

```yml
      - args:
        - --cert-dir=/tmp
        - --secure-port=4443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
```

==>

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
-->

Apply the manifest `components.yaml`:

```dos
git clone https://github.com/briansu2004/udemy-devops-real-projects.git
cd udemy-devops-real-projects\010-MinikubeGrafanaPrometheusMultipassMonitoring
kubectl -n kube-system apply -f components.yaml
```

<!--
```dos
C:\devbox\udemy-devops-real-projects\010-MinikubeGrafanaPrometheusMultipassMonitoring>kubectl -n kube-system apply -f components.yaml
serviceaccount/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
service/metrics-server created
deployment.apps/metrics-server created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
```
-->

<!--
```dos
C:\devbox\udemy-devops-real-projects\010-MinikubeGrafanaPrometheusMultipassMonitoring>kubectl get pods --all-namespaces
NAMESPACE              NAME                                        READY   STATUS    RESTARTS      AGE
kube-system            coredns-787d4945fb-ht7sf                    1/1     Running   1 (18m ago)   10h
kube-system            etcd-minikube                               1/1     Running   1 (18m ago)   10h
kube-system            kube-apiserver-minikube                     1/1     Running   1 (18m ago)   10h
kube-system            kube-controller-manager-minikube            1/1     Running   1 (18m ago)   10h
kube-system            kube-proxy-kl67r                            1/1     Running   1 (18m ago)   10h
kube-system            kube-scheduler-minikube                     1/1     Running   1 (18m ago)   10h
kube-system            metrics-server-986754fb7-ljt59              1/1     Running   0             2m43s
kube-system            storage-provisioner                         1/1     Running   2 (17m ago)   10h
kubernetes-dashboard   dashboard-metrics-scraper-5c6664855-9c46c   1/1     Running   1 (18m ago)   10h
kubernetes-dashboard   kubernetes-dashboard-55c4cbbc7c-mqrzz       1/1     Running   2 (17m ago)   10h
```
-->

Once the Pod is ready, we can run below command to test out if the metric server is working.

```dos
kubectl top node
```

We should be able to see below result if it works fine.

```dos
NAME       CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
minikube   1737m        21%    2380Mi          9%
```

#### 3. Add helm repo

Once Helm is set up properly, **add** the **repo** as follows:

```dos
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm repo update

helm search repo prometheus-community
```

#### 4. Deploy Prometheus Helm chart

Install Prometheus Helm Chart by running below command:

```dos
helm install prometheus-grafana prometheus-community/kube-prometheus-stack -f values.yaml
```

<!--
```dos
C:\devbox\udemy-devops-real-projects\010-MinikubeGrafanaPrometheusMultipassMonitoring>helm install prometheus-grafana prometheus-community/kube-prometheus-stack -f values.yaml
NAME: prometheus-grafana
LAST DEPLOYED: Mon Apr 10 21:04:27 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
kube-prometheus-stack has been installed. Check its status by running:
  kubectl --namespace default get pods -l "release=prometheus-grafana"

Visit https://github.com/prometheus-operator/kube-prometheus for instructions on how to create & configure Alertmanager and Prometheus instances using the Operator.
```
-->

#### 5. Configure Grafana Dashboard manually

Once the deployment is settle, we can **port-forward** to the Grafana service to access the portal from our local:

```dos
kubectl -n default port-forward svc/prometheus-grafana 8888:80
```

Open our **brower** and then type the URL: [http://localhost:8888](http://localhost:8888).

We should see the **Grafana login portal**.

![1681175384791](image/01_YN_WindowsOnly/1681175384791.png)

We can retrieve the **admin password** by running below command in another terminal:

```dos
kubectl get secret prometheus-grafana -o=jsonpath="{.data.admin-password}"|base64 -d
```

<!--
``bash
$ kubectl get secret prometheus-grafana -o=jsonpath="{.data.admin-password}"|base64 -d
changeme
```
-->

Enter the username (**admin**) and the password we got above, then we should be able to log in the Grafana welcome board.

Go to the **Dashboards** section in the left navigation lane, and click **+New dashboard** to open a new dashboard.

Follow below steps to add some **variables** before creating a panel.

a. Click **Dashboard settings**(the gear icon) in the top right

b. Go to **Variables** section

c. Click **Add variable** and add below variables

**Node**

```dos
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
- **Custom all value**: .*
```

Click **Apply** to save the change

![1681175861842](image/01_YN_WindowsOnly/1681175861842.png)

![1681177262433](image/01_YN_WindowsOnly/1681177262433.png)

**Container**

```dos
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
- **Custom all value**: .*
```

Click **Apply** to save the change

![1681175941632](image/01_YN_WindowsOnly/1681175941632.png)

![1681177287025](image/01_YN_WindowsOnly/1681177287025.png)

**Namespace**

```dos
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
- **Custom all value**: .*
```

Click **Apply** to save the change

![1681176033749](image/01_YN_WindowsOnly/1681176033749.png)

![1681177307753](image/01_YN_WindowsOnly/1681177307753.png)

**interval**

```dos
- **Select variable type**: Interval
- **Name**: interval
- Label: <leave it blank>
- Description: Interval
<!-- - **Data source**: Prometheus -->
- **Values**: 1m,10m,30m,1h,6h,12h,1d,7d,14d,30d
- Auto Option: Enable
- Step count: 1
- Min interval: 2m
```

Click **Apply** to save the change

![1681176168568](image/01_YN_WindowsOnly/1681176168568.png)

![1681176233542](image/01_YN_WindowsOnly/1681176233542.png)

Click **Save** to save the dashboard. Name it as **Container Health Status**

![1681176251350](image/01_YN_WindowsOnly/1681176251350.png)

Once we go back to the Dashboard, we will see the **Node**/**Container**/**Namespace**/**Interval** sections are available in the top left with dropdown menu for choosing.

Now we are going to add a new **Panel**.

Click **Add Panel** in the top right and click **Add a new panel** area.

In the section **A**, click "Code" button (right to the "Builder" button) then type below query in the **Enter a PromQL query** field:

```dos
sum(kube_pod_status_phase{pod=~"^$Container.*",namespace=~"default"}) by (phase)
```

and click **Run queries** to execute the query.

![1681177404556](image/01_YN_WindowsOnly/1681177404556.png)

Make sure to choose **All** in top **Container** dropdown menu. We should see a line chart in above display area.

In order to make the graph more readable, we can change the type of charts. Just expanding the **Time series** section in the top right and search for **bar gauge** to apply.

![1681177448749](image/01_YN_WindowsOnly/1681177448749.png)

Before saving the change, go to **Panel options** section in the right lane and type the name in **Title** field, for example, **Pod Status in Default Namespace**. And click **Apply** to save the change.

![1681177503995](image/01_YN_WindowsOnly/1681177503995.png)

Next, we will create a **panel** to monitor the **Top 5 memory intense Pods**.

Again, click **Add panel** and then choose **Add a new panel**.

Copy and paste below query in the query field:

```dos
topk(5,avg(container_memory_usage_bytes{}) by (pod) /1024/1024/1024)
```

and click **Run queries** to execute the query.

Expanding the **Time series** section in the top right and search for **Bar gauge** to apply.

We can also change the layout orientation in **Bar gauge** -> **Orientation** section.

![1681177618885](image/01_YN_WindowsOnly/1681177618885.png)

![1681177657632](image/01_YN_WindowsOnly/1681177657632.png)

<!--
![Top 5 Memory Intense Pods](images/top-5-memory-intense-pod.png)
-->

#### 6. Configure Dashboard by Importing Json file

Instead of manually configuring the dashboard, we can also **import the pre-defined dashboard from a json file**.

In the Grafana Home page, go to **Dashboards** and click **Import**.

![1681177891790](image/01_YN_WindowsOnly/1681177891790.png)

Click **Upload JSON file** and choose **pod-health-status.json**. Then we should see the dashboard imported.

![1681177812192](image/01_YN_WindowsOnly/1681177812192.png)

![1681177824556](image/01_YN_WindowsOnly/1681177824556.png)

#### 7. Download Dashboard Template

A variety of dashboard templates are available to meet different needs in [**Grafana Labs**](https://grafana.com/grafana/dashboards/).

We can go to there and search for any dashboard we like, and just need to copy the **ID** and paste to **Grafana website** -> **Dashboard** -> **Import** -> **Import via grafana.com** and click **Load** to load the template from the website.

![Template ID](images/template-id.png)

![Template Import](images/template-import.png)

![1681178021156](image/01_YN_WindowsOnly/1681178021156.png)

![1681178038563](image/01_YN_WindowsOnly/1681178038563.png)
