# **8 Redis**

[https://github.com/oliver006/redis_exporter](https://github.com/oliver006/redis_exporter)

### **8.1. Redis down**

**Redis instance is down**

```
- alert: RedisDown
  expr: redis_up == 0
  for: 5m
  labels:
    severity: error
  annotations:
    summary: "Redis down (instance {{ $labels.instance }})"
    description: "Redis instance is down\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **8.2. Redis missing master**

**Redis cluster has no node marked as master.**

```
  - alert: RedisMissingMaster
    expr: (count(redis_instance_info{role="master"}) or vector(0)) < 1
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Redis missing master (instance {{ $labels.instance }})
      description: "Redis cluster has no node marked as master.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


### **8.3. Redis too many masters**

**Redis cluster has too many nodes marked as master.**

```
  - alert: RedisTooManyMasters
    expr: count(redis_instance_info{role="master"}) > 1
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Redis too many masters (instance {{ $labels.instance }})
      description: "Redis cluster has too many nodes marked as master.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **8.4. Redis disconnected slaves**

**Redis not replicating for all slaves. Consider reviewing the redis replication status.**

```
  - alert: RedisDisconnectedSlaves
    expr: count without (instance, job) (redis_connected_slaves) - sum without (instance, job) (redis_connected_slaves) - 1 > 1
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Redis disconnected slaves (instance {{ $labels.instance }})
      description: "Redis not replicating for all slaves. Consider reviewing the redis replication status.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **8.5. Redis replication broken**

**Redis instance lost a slave**

```
  - alert: RedisReplicationBroken
    expr: delta(redis_connected_slaves[1m]) < 0
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Redis replication broken (instance {{ $labels.instance }})
      description: "Redis instance lost a slave\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **8.6. Redis cluster flapping**

**Changes have been detected in Redis replica connection. This can occur when replica nodes lose connection to the master and reconnect (a.k.a flapping).**

```
  - alert: RedisClusterFlapping
    expr: changes(redis_connected_slaves[1m]) > 1
    for: 2m
    labels:
      severity: critical
    annotations:
      summary: Redis cluster flapping (instance {{ $labels.instance }})
      description: "Changes have been detected in Redis replica connection. This can occur when replica nodes lose connection to the master and reconnect (a.k.a flapping).\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **8.7. Missing backup**

**Redis has not been backuped for 24 hours**

```
- alert: MissingBackup
  expr: time() - redis_rdb_last_save_timestamp_seconds > 60 * 60 * 24
  for: 5m
  labels:
    severity: error
  annotations:
    summary: "Missing backup (instance {{ $labels.instance }})"
    description: "Redis has not been backuped for 24 hours\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```


### **8.8. Redis out of system memory**

**Redis is running out of memory (> 90%)**

```
- alert: OutOfMemory
  expr: redis_memory_used_bytes / redis_total_system_memory_bytes * 100 > 90
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "Out of memory (instance {{ $labels.instance }})"
    description: "Redis is running out of memory (> 90%)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```


### **8.9. Redis out of configured maxmemory**

**Redis is running out of configured maxmemory (> 90%)**

```
  - alert: RedisOutOfConfiguredMaxmemory
    expr: redis_memory_used_bytes / redis_memory_max_bytes * 100 > 90
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: Redis out of configured maxmemory (instance {{ $labels.instance }})
      description: "Redis is running out of configured maxmemory (> 90%)\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


### **8.10. Too many connections**

**Redis instance has too many connections**

```
- alert: TooManyConnections
  expr: redis_connected_clients > 100
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "Too many connections (instance {{ $labels.instance }})"
    description: "Redis instance has too many connections\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **8.11. Not enough connections**

**Redis instance should have more connections (> 5)**

```
- alert: NotEnoughConnections
  expr: redis_connected_clients < 5
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "Not enough connections (instance {{ $labels.instance }})"
    description: "Redis instance should have more connections (> 5)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **8.12. Rejected connections**

**Some connections to Redis has been rejected**

```
- alert: RejectedConnections
  expr: increase(redis_rejected_connections_total[1m]) > 0
  for: 5m
  labels:
    severity: error
  annotations:
    summary: "Rejected connections (instance {{ $labels.instance }})"
    description: "Some connections to Redis has been rejected\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

