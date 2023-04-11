# Lab 010: Deploy Prometheus/Grafana on Minikube and Monitor The Health of Containers and VMs

## Goal

In this lab, we will deploy the **Prometheus-Grafana** **Helm** chart on **Minikube**, and then set up a **dashboard** to monitor the health of the containers in the Minikube cluster, as well as a VM created by **Multipass**.

## Environments

| #  | Env  | Y/N  | Recommended   |  Comment |
|---|---|---|---|---|
| 1 | Windows only | N | N |   |
| 2 | Windows + Ubuntu | Y | Y |   |
| 3 | Mac only | Y | Y |   |
| 4 | Mac + Ubuntu | Y | Y |   |

[Windows Only](01_Y_WindowsOnly.md)

<!--
[With_Windows_Ubuntu](02_YN_Windows_Ubuntu.md)

[Mac Only](03_Y_MacOnly.md)

[With_Mac_Ubuntu](04_YN_Mac_Ubuntu.md)
-->

<!--
## <a name="troubleshooting">Troubleshooting</a>

### <a name=issue1>Issue 1: Error from server (ServiceUnavailable): the server is currently unable to handle the request (get nodes.metrics.k8s.io)</a>

When deploying metrics server in the cluster, the deployment won't be ready and showing below error

```
E0112 15:02:25.912192       1 scraper.go:140] "Failed to scrape node" err="Get \"https://192.168.49.2:10250/metrics/resource\": x509: cannot validate certificate for 192.168.49.2 because it doesn't contain any IP SANs" node="minikube"
```

When we run `kubectl top node` below error occurs:

```
Error from server (ServiceUnavailable): the server is currently unable to handle the request (get nodes.metrics.k8s.io)
```

Solution:
Add below command section in the deployment manifest yaml file to disable the TLS verification

```
command:
- --kubelet-insecure-tls
- --kubelet-preferred-address-types=InternalIP
```

Above are all steps to deploy/setup Premotheus-Grafana in a Kubernetes cluster.

> ref: <https://thospfuller.com/2020/11/29/easy-kubernetes-metrics-server-install-in-minikube-in-five-steps/>

## <a name="reference">Reference</a>

- [Prometheus Overview](https://prometheus.io/docs/introduction/overview/)
- [Grafana Github README](https://github.com/grafana/helm-charts/blob/main/charts/grafana/README.md)
- [Grafana Awesome Alert](https://awesome-prometheus-alerts.grep.to/)
- [Prometheus Queries Example Official](https://prometheus.io/docs/prometheus/latest/querying/examples/)
- [Prometheus Queries Example 1](https://www.opsramp.com/guides/prometheus-monitoring/prometheus-alerting/)
- [Prometheus Queries Example 2](https://sysdig.com/blog/prometheus-query-examples/)
- [Prometheus Queries Example 3](https://sysdig.com/blog/getting-started-with-promql-cheatsheet/)
- [Prometheus Queries Example 4](https://www.containiq.com/post/promql-cheat-sheet-with-examples)
- [Node Exporter Installation](https://prometheus.io/docs/guides/node-exporter/)
- [Step-by-step guide to setting up Prometheus Alertmanager with Slack, PagerDuty, and Gmail](https://grafana.com/blog/2020/02/25/step-by-step-guide-to-setting-up-prometheus-alertmanager-with-slack-pagerduty-and-gmail/)
- [Alerting Rules](https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/)
- [Defining Recording Rules](https://prometheus.io/docs/prometheus/latest/configuration/recording_rules/)
- [Setup Alertmanager](https://ashish.one/blogs/setup-alertmanager/)
- [Alert Script](https://gist.github.com/cherti/61ec48deaaab7d288c9fcf17e700853a)

```
url=prometheus-2-alertmanager.monitoring.svc:9093/api/v2/alerts
startsAt=`date --iso-8601=seconds`
endsAt=`date --iso-8601=seconds`
curl -XPOST $url -H "Content-Type: application/json" -d '[{"status": "firing","labels": {"alertname": "my_cool_alert","service": "curl","severity": "warning","instance": "0"},"annotations": {"summary": "This is a summary","description": "This is a description."},"generatorURL": "http://prometheus.int.example.net/<generating_expression>","startsAt": "2023-01-15T01:05:36+00:00"}]'

curl -XPOST $url -H "Content-Type: application/json" -d '[{"status": "firing","labels": {"alertname": "my_cool_alert","service": "curl","severity": "warning","instance": "0"},"annotations": {"summary": "This is a summary","description": "This is a description."},"generatorURL": "http://prometheus.int.example.net/<generating_expression>","startsAt": "'`date --iso-8601=seconds`'"}]'

curl -XPOST -H "Content-Type: application/json" $url -d '[{"status": "resolved","labels": {"alertname": "my_cool_alert","service": "curl","severity": "warning","instance": "0"},"annotations": {"summary": "This is a summary","description": "This is a description."},"generatorURL": "http://prometheus.int.example.net/<generating_expression>","startsAt": "2020-07-23T01:05:36+00:00","endsAt": "2020-07-23T01:05:38+00:00"}]'
```

- [Alert Script 2](https://gist.github.com/cherti/61ec48deaaab7d288c9fcf17e700853a)
- [Alert Script 3](https://gist.github.com/carinadigital/fd2960fdccd77dbdabc849656c43a070)
- [stress-ng USAGE](https://stackoverflow.com/questions/45317515/stress-ng-ram-testing-commands)
- [Multipass Commandline](https://multipass.run/docs/launch-command)
-->
