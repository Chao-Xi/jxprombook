# **Chap4 Prometheus Operator Monitor on Rabbitmq**

1. 第一步，安装`rabbitmq-exporter`, `rabbit-ha` 和 `rabbit-transient`
2. 第二步为 `ServiceMonitor` 对象关联 `metrics` 数据接口的一个 `Service` 对象
3. 第三步确保 `Service` 对象可以正确获取到 `metrics` 数据


## **安装`rabbitmq-exporter`**

### **`rabbit-ha rabbitmq-exporter`**

```
helm install prometheus-rabbitmq-exporter-ha --set rabbitmq.url=http://rabbitmq-ha:15672,rabbitmq.user=jambunny,rabbitmq.existingPasswordSecret=rabbit stable/prometheus-rabbitmq-exporter -n $JAM_INSTANCE
```

### `rabbit-transient rabbitmq-exporter`

```
helm install prometheus-rabbitmq-exporter-transient --set rabbitmq.url=http://rabbitmq-transient:15672,rabbitmq.user=jambunny,rabbitmq.existingPasswordSecret=rabbit stable/prometheus-rabbitmq-exporter -n $JAM_INSTANCE
```

* **Name**: `prometheus-rabbitmq-exporter-ha`&`prometheus-rabbitmq-exporter-transient`
* **rabbitmq.url**: `http://rabbitmq-transient:15672`
* **rabbitmq.user**: jambunny
* **rabbitmq.existingPasswordSecret** : rabbit
* **stable/prometheus-rabbitmq-exporter**

```
$ helm list  -n integration701  | grep rab
...
prometheus-rabbitmq-exporter-ha         integration701  1               2020-02-12 16:39:52.9999 +0800 CST      deployed        prometheus-rabbitmq-exporter-0.5.5        v0.29.0    
prometheus-rabbitmq-exporter-transient  integration701  1               2020-02-12 16:40:42.605152 +0800 CST    deployed        prometheus-rabbitmq-exporter-0.5.5        v0.29.0 
```

### **创建 ServiceMonitor** 

**`rabbit-ha.yaml`**

```
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: rabbit-ha
  labels:
    prometheus: kube-prometheus
spec:
  selector:
    # note, this matches on the service, not the deployment or pod
    matchLabels:
      app: prometheus-rabbitmq-exporter
      release: prometheus-rabbitmq-exporter-ha
  jobLabel: app
  endpoints:
  - targetPort: 9419
    path: /metrics
  namespaceSelector:
    matchNames:
      - integration701
```

1. 上面我们在 `default` 命名空间下面创建了名为 `rabbit-ha` 的 `ServiceMonitor` 对象
2. 匹配 `integration701` 这个命名空间下面的具有 `app: prometheus-rabbitmq-exporter` 和 `app: prometheus-rabbitmq-exporter`这两个 `label` 标签的 `Service`，`jobLabel` 表示用于检索 `job` 任务名称的标签，

#### `rabbit-transient.yaml`

```
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: rabbit-transient
  labels:
    prometheus: kube-prometheus
spec:
  selector:
    # note, this matches on the service, not the deployment or pod
    matchLabels:
      app: prometheus-rabbitmq-exporter
      release: prometheus-rabbitmq-exporter-transient
  jobLabel: app
  endpoints:
  - targetPort: 9419
    path: /metrics
  namespaceSelector:
  	matchNames:
   - integration701
```

1. 上面我们在 `default` 命名空间下面创建了名为 `rabbit-transient` 的 `ServiceMonitor` 对象
2. 匹配 `integration701` 这个命名空间下面的具有 `app: prometheus-rabbitmq-exporter` 和 `app: prometheus-rabbitmq-exporter`这两个 `label` 标签的 `Service`，`jobLabel` 表示用于检索 `job` 任务名称的标签，

```
$ kubectl get servicemonitor
NAMESPACE    NAME                         AGE
default      rabbit-ha                    44d
default      rabbit-transient             44d
```

### 查看 `Service`

```
$ kubectl get svc -n integration701 | grep rab
prometheus-rabbitmq-exporter-ha          ClusterIP      100.68.97.158    <none>                                                   
                   9419/TCP                                  44d
prometheus-rabbitmq-exporter-transient   ClusterIP      100.69.144.253   <none>                                                   
                   9419/TCP                                  44d
rabbitmq-exporter-ha                     ClusterIP      100.65.251.106   <none>                                                   
                   9419/TCP                                  66d
rabbitmq-exporter-transient              ClusterIP      100.66.131.184   <none>                                                   
                   9419/TCP                                  66d
rabbitmq-ha                              ClusterIP      100.71.17.211    <none>                                                   
                   15672/TCP,5672/TCP                        65d
rabbitmq-transient                       ClusterIP      100.71.81.171    <none>                                                   
                   15672/TCP,5672/TCP                        65d
```


## **`monitoring/kube-prometheus/prometheus-rules.yaml`**

### **Reference** 

[`alerts.yaml`](https://github.com/helm/charts/blob/master/stable/rabbitmq-ha/templates/alerts.yaml)

### **Jam `rabbitmq-ha-rules`**

`rabbitmq-ha-rules.yaml`

```
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  labels:
    prometheus: k8s
    role: alert-rules
  name: rabbitmq-ha-rules
  namespace: monitoring
spec:
  groups:
  - name: rabbitmq-ha.rules
    rules:
    - alert: RabbitMqHaClusterNodeDown
      expr: rabbitmq_up{service="prometheus-rabbitmq-exporter-ha"} == 0
      for: 5m
      labels:
        severity: critical
        job: rabbitmq-ha
      annotations:
        summary:  "[highly_available] Rabbitmq down (Detected by Service {{$labels.service}} and Exporter {{$labels.pod}})"
        description: "RabbitMQ-HA {{$labels.namespace}}/{{$labels.pod}} is down"
    - alert: RabbitMqHaClusterNotAllNodesRunning
      expr: rabbitmq_running{service="prometheus-rabbitmq-exporter-ha"} < kube_statefulset_replicas{namespace=~"^(jam|integration|stage|dev).*", statefulset=~"rabbitmq-ha"}
      for: 5m
      labels:
        severity: critical
        job: rabbitmq-ha
      annotations:
        summary: "[highly_available]  Some RabbitMQ-HA Cluster Nodes Are Down in Service (Detected by Service {{$labels.service}} and Exporter {{$labels.pod}})"
        description: "Some RabbitMQ-HA cluster nodes are down, currently running with VALUE = {{$value}}"
    - alert: RabbitMqHaClusterPartition
      expr: rabbitmq_partitions{service="prometheus-rabbitmq-exporter-ha"} > 0
      for: 5m
      labels:
        severity: critical
        job: rabbitmq-ha
      annotations:
        summary: "[highly_available] Cluster Partition Error (Detected by Service {{$labels.service}} and Exporter {{$labels.pod}})"
        description: "RabbitMQ-HA either some nodes are down or the cluster has partitioned with VALUE = {{$value}}"
    - alert: RabbitMqHaExchangeStatus
      expr: rabbitmq_exchangesTotal{service="prometheus-rabbitmq-exporter-ha"} == 0
      for: 5m
      labels:
        severity: critical
        job: rabbitmq-ha
      annotations:
        summary: "[highly_available] Cluster Exchange are Missing (Detected by Service {{$labels.service}} and Exporter {{$labels.pod}})"
        description: "The count of RabbitMQ-HA cluster exchange is VALUE = {{$value}}"
    - alert: RabbitMqHaQueueStatus
      expr: rabbitmq_queuesTotal{service="prometheus-rabbitmq-exporter-ha"} == 0
      for: 5m
      labels:
        severity: critical
        job: rabbitmq-ha
      annotations:
        summary: "[highly_available] Cluster Queues are Missing (Detected by Service {{$labels.service}} and Exporter {{$labels.pod}})"
        description: "The count of RabbitMQ-HA cluster queue is VALUE = {{$value}}"
    - alert: RabbitMqHaDiskSpaceAlarm
      expr: rabbitmq_node_disk_free_alarm{service="prometheus-rabbitmq-exporter-ha"} == 1
      for: 5m
      labels:
        severity: critical
        job: rabbitmq-ha
      annotations:
        summary: "[highly_available] RabbitMQ-HA is Out of Disk Space (Detected by Service {{$labels.service}} and Exporter {{$labels.pod}})"
        description: "RabbitMQ-HA {{$labels.namespace}} / {{$labels.pod}} Disk Space Alarm is going off.  Which means the node hit highwater mark and has cut off network connectivity, see RabbitMQ WebUI"
    - alert: RabbitMqHaMemoryAlarm
      expr: rabbitmq_node_mem_alarm{service="prometheus-rabbitmq-exporter-ha"} == 1
      for: 5m
      labels:
        severity: critical
        job: rabbitmq-ha
      annotations:
        summary: "[highly_available] RabbitMQ-HA is Out of Memory (Detected by Service {{$labels.service}} and Exporter {{$labels.pod}})"
        description: "RabbitMQ-HA {{$labels.namespace}} / {{$labels.pod}} High Memory Alarm is going off.  Which means the node hit highwater mark and has cut off network connectivity, see RabbitMQ WebUI"
    - alert: RabbitMqHaMemoryUsageHigh
      expr: (rabbitmq_node_mem_used{service="prometheus-rabbitmq-exporter-ha"} / rabbitmq_node_mem_limit{service="prometheus-rabbitmq-exporter-ha"} ) * 100 > 90
      for: 5m
      labels:
        severity: critical
        job: rabbitmq-ha
      annotations:
        summary: "[highly_available] RabbitMQ-HA Node > 90% Memory Usage (Detected by Service {{$labels.service}} and Exporter {{$labels.pod}})"
        description: "RabbitMQ-HA {{$labels.namespace}} / {{$labels.pod}}  Memory Usage > 90%"
    - alert: RabbitMqHaFileDescriptorsLow
      expr: (rabbitmq_fd_used{service="prometheus-rabbitmq-exporter-ha"} / rabbitmq_fd_total{service="prometheus-rabbitmq-exporter-ha"}) * 100 > 90
      for: 5m
      labels:
        severity: critical
        job: rabbitmq-ha
      annotations:
        summary: "RabbitMQ-HA Low File Descriptor Available (Detected by Service {{$labels.service}} and Exporter {{$labels.pod}})"
        description: "RabbitMQ-HA {{$labels.namespace}} / {{$labels.pod}} File Descriptor Usage > 90%"
    - alert: RabbitMqHaDiskSpaceLow
      expr: predict_linear(rabbitmq_node_disk_free{service="prometheus-rabbitmq-exporter-ha"}[15m], 1 * 60 * 60) < rabbitmq_node_disk_free_limit{service="prometheus-rabbitmq-exporter-ha"}
      for: 5m
      labels:
        severity: critical
        job: rabbitmq-ha
      annotations:
        summary: "[highly_available] RabbitMQ-HA is Low on Disk Space and will Run Out in the next hour (Detected by Service {{$labels.service}} and Exporter {{$labels.pod}})"
        description: "RabbitMQ-HA {{$labels.namespace}} / {{$labels.pod}} will hit disk limit in the next hr based on last 15 mins trend."
    - alert: RabbitMqHaNoConsumer
      expr: rabbitmq_queue_consumers{service="prometheus-rabbitmq-exporter-ha"} == 0
      for: 5m
      labels:
        severity: critical
        job: rabbitmq-ha
      annotations:
        summary: "[highly_available] No Consumer Existing for Queues in RabbitMQ-HA  (instance {{$labels.instance}})"
        description: "RabbitMQ-HA Queues has no consumers exist for {{$labels.queue}}"
```


