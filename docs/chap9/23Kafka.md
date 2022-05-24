# 23. Kafka : `kafka_exporter`

## **1 Kafka [`danielqsj/kafka_exporter `](https://github.com/danielqsj/kafka_exporter)**

### **1. Kafka Topics**

**Kafka topic in-sync partition**

```
- alert: KafkaTopics
  expr: sum(kafka_topic_partition_in_sync_replica) by (topic) < 3
  for: 5m
  labels:
    severity: error
  annotations:
    summary: "Kafka Topics (instance {{ $labels.instance }})"
    description: "Kafka topic in-sync partition\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **2. Kafka consumers group**

**Kafka consumers group**

```
- alert: KafkaConsumersGroup
  expr: sum(kafka_consumergroup_lag) by (consumergroup) > 50
  for: 5m
  labels:
    severity: error
  annotations:
    summary: "Kafka consumers group (instance {{ $labels.instance }})"
    description: "Kafka consumers group\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

## **2 Kafka : [linkedin/Burrow](https://github.com/linkedin/Burrow)**

### **1. Kafka topic offset decreased**

**Kafka topic offset has decreased**

```
  - alert: KafkaTopicOffsetDecreased
    expr: delta(kafka_burrow_partition_current_offset[1m]) < 0
    for: 0m
    labels:
      severity: warning
    annotations:
      summary: Kafka topic offset decreased (instance {{ $labels.instance }})
      description: "Kafka topic offset has decreased\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


### **2. Kafka consumer lag**

**Kafka consumer has a 30 minutes and increasing lag**

```
  - alert: KafkaConsumerLag
    expr: kafka_burrow_topic_partition_offset - on(partition, cluster, topic) group_right() kafka_burrow_partition_current_offset >= (kafka_burrow_topic_partition_offset offset 15m - on(partition, cluster, topic) group_right() kafka_burrow_partition_current_offset offset 15m) AND kafka_burrow_topic_partition_offset - on(partition, cluster, topic) group_right() kafka_burrow_partition_current_offset > 0
    for: 15m
    labels:
      severity: warning
    annotations:
      summary: Kafka consumer lag (instance {{ $labels.instance }})
      description: "Kafka consumer has a 30 minutes and increasing lag\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```
