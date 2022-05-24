# 8. Elasticsearch: `elasticsearch_exporter`

[https://github.com/justwatchcom/elasticsearch_exporter](https://github.com/justwatchcom/elasticsearch_exporter)

### **1. Elastic Heap Usage Too High**

**The heap usage is over 90% for 5m**

```
- alert: ElasticHeapUsageTooHigh
  expr: (elasticsearch_jvm_memory_used_bytes{area="heap"} / elasticsearch_jvm_memory_max_bytes{area="heap"}) * 100 > 90
  for: 5m
  labels:
    severity: error
  annotations:
    summary: "Elastic Heap Usage Too High (instance {{ $labels.instance }})"
    description: "The heap usage is over 90% for 5m\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **2. Elastic Heap Usage warning**

**The heap usage is over 80% for 5m**

```
- alert: ElasticHeapUsageWarning
  expr: (elasticsearch_jvm_memory_used_bytes{area="heap"} / elasticsearch_jvm_memory_max_bytes{area="heap"}) * 100 > 80
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "Elastic Heap Usage warning (instance {{ $labels.instance }})"
    description: "The heap usage is over 80% for 5m\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **3. Elasticsearch disk out of space**

**The disk usage is over 90%**

```
  - alert: ElasticsearchDiskOutOfSpace
    expr: elasticsearch_filesystem_data_available_bytes / elasticsearch_filesystem_data_size_bytes * 100 < 10
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Elasticsearch disk out of space (instance {{ $labels.instance }})
      description: "The disk usage is over 90%\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **4. Elasticsearch disk space low**

The disk usage is over 80%

```
  - alert: ElasticsearchDiskSpaceLow
    expr: elasticsearch_filesystem_data_available_bytes / elasticsearch_filesystem_data_size_bytes * 100 < 20
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: Elasticsearch disk space low (instance {{ $labels.instance }})
      description: "The disk usage is over 80%\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

```

### **5. Elastic Cluster Red**

**Elastic Cluster Red status**

```
- alert: ElasticClusterRed
  expr: elasticsearch_cluster_health_status{color="red"} == 1
  for: 5m
  labels:
    severity: error
  annotations:
    summary: "Elastic Cluster Red (instance {{ $labels.instance }})"
    description: "Elastic Cluster Red status\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **6. Elastic Cluster Yellow**

**Elastic Cluster Yellow status**

```
- alert: ElasticClusterYellow
  expr: elasticsearch_cluster_health_status{color="yellow"} == 1
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "Elastic Cluster Yellow (instance {{ $labels.instance }})"
    description: "Elastic Cluster Yellow status\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **7. Elasticsearch Healthy Nodes**

**Missing node in Elasticsearch cluster**

```
  - alert: ElasticsearchHealthyNodes
    expr: elasticsearch_cluster_health_number_of_nodes < 3
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Elasticsearch Healthy Nodes (instance {{ $labels.instance }})
      description: "Missing node in Elasticsearch cluster\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


### **8. Number of Elastic Healthy Nodes**

**Number Healthy Nodes less then `number_of_nodes`**

```
- alert: NumberOfElasticHealthyNodes
  expr: elasticsearch_cluster_health_number_of_nodes < number_of_nodes
  for: 5m
  labels:
    severity: error
  annotations:
    summary: "Number of Elastic Healthy Nodes (instance {{ $labels.instance }})"
    description: "Number Healthy Nodes less then number_of_nodes\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **9. Elasticsearch relocating shards**

**Elasticsearch is relocating shards**

```
  - alert: ElasticsearchRelocatingShards
    expr: elasticsearch_cluster_health_relocating_shards > 0
    for: 0m
    labels:
      severity: info
    annotations:
      summary: Elasticsearch relocating shards (instance {{ $labels.instance }})
      description: "Elasticsearch is relocating shards\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **10. Elasticsearch relocating shards too long**

**Elasticsearch has been relocating shards for 15min**

```
    - alert: ElasticsearchRelocatingShardsTooLong
    expr: elasticsearch_cluster_health_relocating_shards > 0
    for: 15m
    labels:
      severity: warning
    annotations:
      summary: Elasticsearch relocating shards too long (instance {{ $labels.instance }})
      description: "Elasticsearch has been relocating shards for 15min\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **11. Elasticsearch initializing shards**

**Elasticsearch is initializing shards**

```
  - alert: ElasticsearchInitializingShards
    expr: elasticsearch_cluster_health_initializing_shards > 0
    for: 0m
    labels:
      severity: info
    annotations:
      summary: Elasticsearch initializing shards (instance {{ $labels.instance }})
      description: "Elasticsearch is initializing shards\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **12. Elasticsearch initializing shards too long**

**Elasticsearch has been initializing shards for 15 min**

```
  - alert: ElasticsearchInitializingShardsTooLong
    expr: elasticsearch_cluster_health_initializing_shards > 0
    for: 15m
    labels:
      severity: warning
    annotations:
      summary: Elasticsearch initializing shards too long (instance {{ $labels.instance }})
      description: "Elasticsearch has been initializing shards for 15 min\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **13. Elasticsearch unassigned shards**

**Elasticsearch has unassigned shards**

```
  - alert: ElasticsearchUnassignedShards
    expr: elasticsearch_cluster_health_unassigned_shards > 0
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Elasticsearch unassigned shards (instance {{ $labels.instance }})
      description: "Elasticsearch has unassigned shards\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


### **14. Elasticsearch pending tasks**

**Elasticsearch has pending tasks. Cluster works slowly.**

```
  - alert: ElasticsearchPendingTasks
    expr: elasticsearch_cluster_health_number_of_pending_tasks > 0
    for: 15m
    labels:
      severity: warning
    annotations:
      summary: Elasticsearch pending tasks (instance {{ $labels.instance }})
      description: "Elasticsearch has pending tasks. Cluster works slowly.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


### **15 Elasticsearch no new documents**

**No new documents for 10 min!**

```
  - alert: ElasticsearchNoNewDocuments
    expr: increase(elasticsearch_indices_docs{es_data_node="true"}[10m]) < 1
    for: 0m
    labels:
      severity: warning
    annotations:
      summary: Elasticsearch no new documents (instance {{ $labels.instance }})
      description: "No new documents for 10 min!\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


### **16. Number of pending tasks**

**Number of pending tasks for 10 min. Cluster works slowly.**

```
- alert: NumberOfPendingTasks
  expr: elasticsearch_cluster_health_number_of_pending_tasks > 0
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "Number of pending tasks (instance {{ $labels.instance }})"
    description: "Number of pending tasks for 10 min. Cluster works slowly.\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **17. Number of Elastic Healthy Data Nodes**

**Number Healthy Data Nodes less then `number_of_data_nodes`**

```
- alert: NumberOfElasticHealthyDataNodes
  expr: elasticsearch_cluster_health_number_of_data_nodes < number_of_data_nodes
  for: 5m
  labels:
    severity: error
  annotations:
    summary: "Number of Elastic Healthy Data Nodes (instance {{ $labels.instance }})"
    description: "Number Healthy Data Nodes less then number_of_data_nodes\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```
