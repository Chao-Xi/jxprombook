# **20. Consul : prometheus/consul_exporter**

[Consul Exporter](https://github.com/prometheus/consul_exporter)

### **1. Service healthcheck failed**

* **Service: `{{ $labels.service_name }}`** 
* **Healthcheck: `{{ $labels.service_id }}`**

```
- alert: ServiceHealthcheckFailed
  expr: consul_catalog_service_node_healthy == 0
  for: 5m
  labels:
    severity: error
  annotations:
    summary: "Service healthcheck failed (instance {{ $labels.instance }})"
    description: "Service: `{{ $labels.service_name }}` Healthcheck: `{{ $labels.service_id }}`\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **22. Missing Consul master node**

**Numbers of consul raft peers less then expected` <https://example.ru/ui/{{ $labels.dc }}/services/consul|Consul masters>`**

```
- alert: MissingConsulMasterNode
  expr: consul_raft_peers < number_of_consul_master
  for: 5m
  labels:
    severity: error
  annotations:
    summary: "Missing Consul master node (instance {{ $labels.instance }})"
    description: "Numbers of consul raft peers less then expected <https://example.ru/ui/{{ $labels.dc }}/services/consul|Consul masters>\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **3. Consul agent unhealthy**

**A Consul agent is down**

```
  - alert: ConsulAgentUnhealthy
    expr: consul_health_node_status{status="critical"} == 1
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Consul agent unhealthy (instance {{ $labels.instance }})
      description: "A Consul agent is down\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

