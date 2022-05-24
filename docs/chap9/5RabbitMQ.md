# 7. RabbitMQ 

## **[rabbitmq/rabbitmq-prometheus](https://github.com/rabbitmq/rabbitmq-prometheus)**

### **1. Rabbitmq node down**

**Less than 3 nodes running in RabbitMQ cluster**

```
  - alert: RabbitmqNodeDown
    expr: sum(rabbitmq_build_info) < 3
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Rabbitmq node down (instance {{ $labels.instance }})
      description: "Less than 3 nodes running in RabbitMQ cluster\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **2. Rabbitmq node not distributed**

**Distribution link state is not 'up'**

```
  - alert: RabbitmqNodeNotDistributed
    expr: erlang_vm_dist_node_state < 3
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Rabbitmq node not distributed (instance {{ $labels.instance }})
      description: "Distribution link state is not 'up'\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **3. Rabbitmq instances different versions**

**Running different version of Rabbitmq in the same cluster, can lead to failure.**

```
  - alert: RabbitmqInstancesDifferentVersions
    expr: count(count(rabbitmq_build_info) by (rabbitmq_version)) > 1
    for: 1h
    labels:
      severity: warning
    annotations:
      summary: Rabbitmq instances different versions (instance {{ $labels.instance }})
      description: "Running different version of Rabbitmq in the same cluster, can lead to failure.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **4. Rabbitmq memory high**

**A node use more than 90% of allocated RAM**

```
  - alert: RabbitmqMemoryHigh
    expr: rabbitmq_process_resident_memory_bytes / rabbitmq_resident_memory_limit_bytes * 100 > 90
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: Rabbitmq memory high (instance {{ $labels.instance }})
      description: "A node use more than 90% of allocated RAM\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **5. Rabbitmq file descriptors usage**

**A node use more than 90% of file descriptors**

```
  - alert: RabbitmqFileDescriptorsUsage
    expr: rabbitmq_process_open_fds / rabbitmq_process_max_fds * 100 > 90
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: Rabbitmq file descriptors usage (instance {{ $labels.instance }})
      description: "A node use more than 90% of file descriptors\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **6. Rabbitmq too many unack messages**

**Too many unacknowledged messages**

```
  - alert: RabbitmqTooManyUnackMessages
    expr: sum(rabbitmq_queue_messages_unacked) BY (queue) > 1000
    for: 1m
    labels:
      severity: warning
    annotations:
      summary: Rabbitmq too many unack messages (instance {{ $labels.instance }})
      description: "Too many unacknowledged messages\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **7 Rabbitmq too many connections**

**The total connections of a node is too high**

```
  - alert: RabbitmqTooManyConnections
    expr: rabbitmq_connections > 1000
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: Rabbitmq too many connections (instance {{ $labels.instance }})
      description: "The total connections of a node is too high\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **8 Rabbitmq no queue consumer**

**A queue has less than 1 consumer**

```
  - alert: RabbitmqNoQueueConsumer
    expr: rabbitmq_queue_consumers < 1
    for: 1m
    labels:
      severity: warning
    annotations:
      summary: Rabbitmq no queue consumer (instance {{ $labels.instance }})
      description: "A queue has less than 1 consumer\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **9. Rabbitmq unroutable messages**

**A queue has unroutable messages**

```
  - alert: RabbitmqUnroutableMessages
    expr: increase(rabbitmq_channel_messages_unroutable_returned_total[1m]) > 0 or increase(rabbitmq_channel_messages_unroutable_dropped_total[1m]) > 0
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: Rabbitmq unroutable messages (instance {{ $labels.instance }})
      description: "A queue has unroutable messages\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


## **2 RabbitMQ :  [kbudde/rabbitmq-exporter](https://github.com/kbudde/rabbitmq_exporter)**

### **10.1. Rabbitmq node down**

**RabbitMQ node down**

```
- alert: RabbitmqDown
  expr: rabbitmq_up == 0
  for: 5m
  labels:
    severity: error
  annotations:
    summary: "Rabbitmq down (instance {{ $labels.instance }})"
    description: "RabbitMQ node down\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **10.2. Rabbitmq Cluster down**

**Less than 3 nodes running in RabbitMQ cluster**

```
- alert: ClusterDown
  expr: rabbitmq_running < 3
  for: 5m
  labels:
    severity: error
  annotations:
    summary: "Cluster down (instance {{ $labels.instance }})"
    description: "Less than 3 nodes running in RabbitMQ cluster\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **10.3. Rabbitmq cluster partition**

**Cluster partition**

```
- alert: ClusterPartition
  expr: rabbitmq_partitions > 0
  for: 5m
  labels:
    severity: error
  annotations:
    summary: "Cluster partition (instance {{ $labels.instance }})"
    description: "Cluster partition\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **10.4. Rabbitmq out of memory**

**Memory available for RabbmitMQ is low (< 10%)**

```
- alert: OutOfMemory
  expr: rabbitmq_node_mem_used / rabbitmq_node_mem_limit * 100 > 90
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "Out of memory (instance {{ $labels.instance }})"
    description: "Memory available for RabbmitMQ is low (< 10%)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```
 
 
### **10.5. Too many connections**

**RabbitMQ instance has too many connections (> 1000)**

```
- alert: TooManyConnections
  expr: rabbitmq_connectionsTotal > 1000
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "Too many connections (instance {{ $labels.instance }})"
    description: "RabbitMQ instance has too many connections (> 1000)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **10.6. Dead letter queue filling up**

**Dead letter queue is filling up (> 10 msgs)**

```
- alert: DeadLetterQueueFillingUp
  expr: rabbitmq_queue_messages{queue="my-dead-letter-queue"} > 10
  for: 5m
  labels:
    severity: error
  annotations:
    summary: "Dead letter queue filling up (instance {{ $labels.instance }})"
    description: "Dead letter queue is filling up (> 10 msgs)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **10.7. Too many messages in queue**

**Queue is filling up (> 1000 msgs)**

```
- alert: TooManyMessagesInQueue
  expr: rabbitmq_queue_messages_ready{queue="my-queue"} > 1000
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "Too many messages in queue (instance {{ $labels.instance }})"
    description: "Queue is filling up (> 1000 msgs)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **10.8. Rabbitmq Slow queue consuming**

**Queue messages are consumed slowly (> 60s)**

```
- alert: SlowQueueConsuming
  expr: time() - rabbitmq_queue_head_message_timestamp{queue="my-queue"} > 60
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "Slow queue consuming (instance {{ $labels.instance }})"
    description: "Queue messages are consumed slowly (> 60s)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **10.9. No consumer**

**Queue has no consumer**

```
- alert: NoConsumer
  expr: rabbitmq_queue_consumers == 0
  for: 5m
  labels:
    severity: error
  annotations:
    summary: "No consumer (instance {{ $labels.instance }})"
    description: "Queue has no consumer\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **10.10. Too many consumers**

**Queue should have only 1 consumer**

```
- alert: TooManyConsumers
  expr: rabbitmq_queue_consumers > 1
  for: 5m
  labels:
    severity: error
  annotations:
    summary: "Too many consumers (instance {{ $labels.instance }})"
    description: "Queue should have only 1 consumer\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **10.11. Unactive exchange**

**Exchange receive less than 5 msgs per second**

```
- alert: UnactiveExchange
  expr: rate(rabbitmq_exchange_messages_published_in_total{exchange="my-exchange"}[1m]) < 5
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "Unactive exchange (instance {{ $labels.instance }})"
    description: "Exchange receive less than 5 msgs per second\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

