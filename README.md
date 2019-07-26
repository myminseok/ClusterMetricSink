# ClusterMetricsSink 

```
<kube-state-metrics(pod)>  <== <ClusterMetricsSink(pod)> <== <Prometheus Server> <== <Grafana>

```

## installation

### kube-state-metrics
- it comes with k8s cluster out of box which is create by PKS if you enabled 'wavefront integration' PKS tile.(wavefront uses kube-state-metrics https://docs.wavefront.com/kubernetes.html)
- otherwise you can install manually following https://github.com/kubernetes/kube-state-metrics/tree/master/kubernetes
```
kubectl apply -f kube-state-metrics-service-account.yaml
kubectl apply -f kube-state-metrics-cluster-role.yaml
kubectl apply -f kube-state-metrics-cluster-role-binding.yaml
kubectl apply -f kube-state-metrics-deployment.yaml
kubectl apply -f kube-state-metrics-service.yaml

kubectl get events -n kube-system

```

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
 ssh into opsman VM. and verify metrics is collected from ClusterMetricSink. 

```
curl -k http://Foo:Bar@<K8s_worker_node_ip>:9273/metrics | grep kube_pod_info
kube_pod_info{cluster_name="mkim-small2",created_by_kind="ReplicaSet",created_by_name="telemetry-agent-776d45f8d8",host="dfcf02e9-fc29-4a52-8529-6d9bc40e8d8b",host_ip="10.10.14.53",namespace="pks-system",node="dfcf02e9-fc29-4a52-8529-6d9bc40e8d8b",pod="telemetry-agent-776d45f8d8-wl57x",pod_ip="10.200.57.8",priority_class="",uid="a67b1774-af5d-11e9-b4e1-00505696ee6a",url="http://10.100.200.201:8080/metrics"} 1

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
  
[![screenshot](/grafana.png)]

