# ClusterMetricsSink 

```
<kube-state-metrics(pod)>  <== <ClusterMetricsSink(pod)> <== <Prometheus Server> <== <Grafana>

```

## installation

### kube-state-metrics
it comes with k8s cluster out of box which is create by PKS, in pks-system namespace.

### deploy ClusterMetricSink
deploy ClusterMetricSink to each k8s cluster.
```
apiVersion: apps.pivotal.io/v1beta1
kind: ClusterMetricSink
metadata:
  name: promcloudsink
spec:
  inputs:
  - type: prometheus
    #urls: ["http://kube-state-metrics.pks-system.svc.cluster.local:8080/metrics"]
    urls: ["http://<YOUR-service-IP>:8080/metrics"]
  outputs:
  - type: prometheus_client
    listen: ":9273"
    basic_username: "Foo"
    basic_password: "Bar"
    path: "/metrics"

```
- replace <YOUR-svc-CLUSTER-IP> with CLUSTER-IP of kube-state-metrics 
  
```
$kubectl get svc --all-namespaces
NAMESPACE     NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        
pks-system    kube-state-metrics     ClusterIP   10.100.200.14    <none>        8080/TCP,8081/TCP   
```
  
deploy to k8s cluster.

```
kubectl apply -f ClusterMetricSink.yml -n pks-system
```

### install Reliability View for PCF
- Prometheus Server, Grafana will deployed by Reliability View for PCF.
- setup scrape configutation on 'Additional Scrape Config Jobs' under TSDB configuration section.
```
- job_name: <unique name for display>
  metrics_path: /metrics
  scheme: http
  basic_auth:
    username: Foo
    password: Bar
  static_configs:
    - targets:
      - "<K8s_worker_node_ip>:9273" 

- job_name: <unique name for display>
  metrics_path: /metrics
  scheme: http
  basic_auth:
    username: Foo
    password: Bar
  static_configs:
    - targets:
      - "<K8s_worker_node_ip>:9273" 


```      
- replase <unique name for display>
- replace <K8s_worker_node_ip> with real IP.


## setup grafana
- WARNING: dashboard template is just draft and should be modified or configured before using.
- import grafana dashboard from <THIS REPO>ClusterMetricSink>granfa>xxx.json via grafana UI 
- prometheus server will automatically collect metrics from ClusterMetricSink in each k8s cluster.


