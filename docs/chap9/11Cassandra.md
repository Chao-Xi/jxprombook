# 9.Cassandra `cassandra_exporter`

## 1.[instaclustr/cassandra_exporter](https://github.com/instaclustr/cassandra-exporter)

### **9.1. Cassandra Node is unavailable**

**Cassandra Node is unavailable - `{{ $labels.cassandra_cluster }} {{ $labels.exported_endpoint }}`**

```
  - alert: CassandraNodeIsUnavailable
    expr: sum(cassandra_endpoint_active) by (cassandra_cluster,instance,exported_endpoint) < 1
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Cassandra Node is unavailable (instance {{ $labels.instance }})
      description: "Cassandra Node is unavailable - {{ $labels.cassandra_cluster }} {{ $labels.exported_endpoint }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **9.2 Cassandra many compaction tasks are pending**

**Many Cassandra compaction tasks are pending - `{{ $labels.cassandra_cluster }}`**

```
  - alert: CassandraManyCompactionTasksArePending
    expr: cassandra_table_estimated_pending_compactions > 100
    for: 0m
    labels:
      severity: warning
    annotations:
      summary: Cassandra many compaction tasks are pending (instance {{ $labels.instance }})
      description: "Many Cassandra compaction tasks are pending - {{ $labels.cassandra_cluster }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **9.3 Cassandra commitlog pending tasks**

**Cassandra commitlog pending tasks - `{{ $labels.cassandra_cluster }}`**

```
  - alert: CassandraCommitlogPendingTasks
    expr: cassandra_commit_log_pending_tasks > 15
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: Cassandra commitlog pending tasks (instance {{ $labels.instance }})
      description: "Cassandra commitlog pending tasks - {{ $labels.cassandra_cluster }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **9.4 Cassandra compaction executor blocked tasks**

Some Cassandra compaction executor tasks are blocked - `{{ $labels.cassandra_cluster }}`

```
  - alert: CassandraCompactionExecutorBlockedTasks
    expr: cassandra_thread_pool_blocked_tasks{pool="CompactionExecutor"} > 15
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: Cassandra compaction executor blocked tasks (instance {{ $labels.instance }})
      description: "Some Cassandra compaction executor tasks are blocked - {{ $labels.cassandra_cluster }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **9.5 Cassandra flush writer blocked tasks**

Some Cassandra flush writer tasks are blocked - `{{ $labels.cassandra_cluster }}`

```
  - alert: CassandraFlushWriterBlockedTasks
    expr: cassandra_thread_pool_blocked_tasks{pool="MemtableFlushWriter"} > 15
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: Cassandra flush writer blocked tasks (instance {{ $labels.instance }})
      description: "Some Cassandra flush writer tasks are blocked - {{ $labels.cassandra_cluster }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **9.6 Cassandra connection timeouts total**

**Some connection between nodes are ending in timeout - `{{ $labels.cassandra_cluster }}`**


```
  - alert: CassandraConnectionTimeoutsTotal
    expr: avg(cassandra_client_request_timeouts_total) by (cassandra_cluster,instance) > 5
    for: 2m
    labels:
      severity: critical
    annotations:
      summary: Cassandra connection timeouts total (instance {{ $labels.instance }})
      description: "Some connection between nodes are ending in timeout - {{ $labels.cassandra_cluster }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **9.7. Cassandra storage exceptions**

**Something is going wrong with cassandra storage - `{{ $labels.cassandra_cluster }}`**

```
  - alert: CassandraStorageExceptions
    expr: changes(cassandra_storage_exceptions_total[1m]) > 1
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Cassandra storage exceptions (instance {{ $labels.instance }})
      description: "Something is going wrong with cassandra storage - {{ $labels.cassandra_cluster }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **9.8. Cassandra tombstone dump**

Cassandra tombstone dump - `{{ $labels.cassandra_cluster }}`

```
  - alert: CassandraTombstoneDump
    expr: avg(cassandra_table_tombstones_scanned{quantile="0.99"}) by (instance,cassandra_cluster,keyspace) > 100
    for: 2m
    labels:
      severity: critical
    annotations:
      summary: Cassandra tombstone dump (instance {{ $labels.instance }})
      description: "Cassandra tombstone dump - {{ $labels.cassandra_cluster }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


### **9.9 Cassandra client request unvailable write**

Some Cassandra client requests are unvailable to write - `{{ $labels.cassandra_cluster }}`

```
  - alert: CassandraClientRequestUnvailableWrite
    expr: changes(cassandra_client_request_unavailable_exceptions_total{operation="write"}[1m]) > 0
    for: 2m
    labels:
      severity: critical
    annotations:
      summary: Cassandra client request unvailable write (instance {{ $labels.instance }})
      description: "Some Cassandra client requests are unvailable to write - {{ $labels.cassandra_cluster }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


### **9.10 Cassandra client request unvailable read**

Some Cassandra client requests are unvailable to read - `{{ $labels.cassandra_cluster }}`

```
  - alert: CassandraClientRequestUnvailableRead
    expr: changes(cassandra_client_request_unavailable_exceptions_total{operation="read"}[1m]) > 0
    for: 2m
    labels:
      severity: critical
    annotations:
      summary: Cassandra client request unvailable read (instance {{ $labels.instance }})
      description: "Some Cassandra client requests are unvailable to read - {{ $labels.cassandra_cluster }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **9.11. Cassandra client request write failure**

Read failures have occurred, ensure there are not too many unavailable nodes - `{{ $labels.cassandra_cluster }}`

```
  - alert: CassandraClientRequestWriteFailure
    expr: increase(cassandra_client_request_failures_total{operation="write"}[1m]) > 0
    for: 2m
    labels:
      severity: critical
    annotations:
      summary: Cassandra client request write failure (instance {{ $labels.instance }})
      description: "Read failures have occurred, ensure there are not too many unavailable nodes - {{ $labels.cassandra_cluster }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **9.12. Cassandra client request read failure**

Read failures have occurred, ensure there are not too many unavailable nodes - `{{ $labels.cassandra_cluster }}`

```
  - alert: CassandraClientRequestReadFailure
    expr: increase(cassandra_client_request_failures_total{operation="read"}[1m]) > 0
    for: 2m
    labels:
      severity: critical
    annotations:
      summary: Cassandra client request read failure (instance {{ $labels.instance }})
      description: "Read failures have occurred, ensure there are not too many unavailable nodes - {{ $labels.cassandra_cluster }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

## **2. Cassandra : [criteo/cassandra_exporter](https://github.com/criteo/cassandra_exporter)**

### **1. Cassandra hints count**

**Cassandra hints count has changed on `{{ $labels.instance }}` some nodes may go down**

```
  - alert: CassandraHintsCount
    expr: changes(cassandra_stats{name="org:apache:cassandra:metrics:storage:totalhints:count"}[1m]) > 3
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Cassandra hints count (instance {{ $labels.instance }})
      description: "Cassandra hints count has changed on {{ $labels.instance }} some nodes may go down\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **2 Cassandra compaction task pending**

**Many Cassandra compaction tasks are pending. You might need to increase I/O capacity by adding nodes to the cluster.**

```
  - alert: CassandraCompactionTaskPending
    expr: avg_over_time(cassandra_stats{name="org:apache:cassandra:metrics:compaction:pendingtasks:value"}[1m]) > 100
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: Cassandra compaction task pending (instance {{ $labels.instance }})
      description: "Many Cassandra compaction tasks are pending. You might need to increase I/O capacity by adding nodes to the cluster.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **3. Cassandra viewwrite latency**

**High viewwrite latency on `{{ $labels.instance }}` cassandra node**

```
  - alert: CassandraViewwriteLatency
    expr: cassandra_stats{name="org:apache:cassandra:metrics:clientrequest:viewwrite:viewwritelatency:99thpercentile",service="cas"} > 100000
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: Cassandra viewwrite latency (instance {{ $labels.instance }})
      description: "High viewwrite latency on {{ $labels.instance }} cassandra node\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **4. Cassandra bad hacker**

**Increase of Cassandra authentication failures**

```
  - alert: CassandraBadHacker
    expr: rate(cassandra_stats{name="org:apache:cassandra:metrics:client:authfailure:count"}[1m]) > 5
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: Cassandra bad hacker (instance {{ $labels.instance }})
      description: "Increase of Cassandra authentication failures\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **5. Cassandra node down**

**Cassandra node down**

```
  - alert: CassandraNodeDown
    expr: sum(cassandra_stats{name="org:apache:cassandra:net:failuredetector:downendpointcount"}) by (service,group,cluster,env) > 0
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Cassandra node down (instance {{ $labels.instance }})
      description: "Cassandra node down\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **6. Cassandra commitlog pending tasks**

**Unexpected number of Cassandra commitlog pending tasks**

```
  - alert: CassandraCommitlogPendingTasks
    expr: cassandra_stats{name="org:apache:cassandra:metrics:commitlog:pendingtasks:value"} > 15
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: Cassandra commitlog pending tasks (instance {{ $labels.instance }})
      description: "Unexpected number of Cassandra commitlog pending tasks\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **7. Cassandra compaction executor blocked tasks**

**Some Cassandra compaction executor tasks are blocked**

```
  - alert: CassandraCompactionExecutorBlockedTasks
    expr: cassandra_stats{name="org:apache:cassandra:metrics:threadpools:internal:compactionexecutor:currentlyblockedtasks:count"} > 0
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: Cassandra compaction executor blocked tasks (instance {{ $labels.instance }})
      description: "Some Cassandra compaction executor tasks are blocked\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **8. Cassandra flush writer blocked tasks**

**Some Cassandra flush writer tasks are blocked**

```
  - alert: CassandraFlushWriterBlockedTasks
    expr: cassandra_stats{name="org:apache:cassandra:metrics:threadpools:internal:memtableflushwriter:currentlyblockedtasks:count"} > 0
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: Cassandra flush writer blocked tasks (instance {{ $labels.instance }})
      description: "Some Cassandra flush writer tasks are blocked\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

###**9. Cassandra repair pending tasks**

**Some Cassandra repair tasks are pending**

```
  - alert: CassandraRepairPendingTasks
    expr: cassandra_stats{name="org:apache:cassandra:metrics:threadpools:internal:antientropystage:pendingtasks:value"} > 2
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: Cassandra repair pending tasks (instance {{ $labels.instance }})
      description: "Some Cassandra repair tasks are pending\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **10. Cassandra repair blocked tasks**

**Some Cassandra repair tasks are blocked**

```
  - alert: CassandraRepairBlockedTasks
    expr: cassandra_stats{name="org:apache:cassandra:metrics:threadpools:internal:antientropystage:currentlyblockedtasks:count"} > 0
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: Cassandra repair blocked tasks (instance {{ $labels.instance }})
      description: "Some Cassandra repair tasks are blocked\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **11. Cassandra connection timeouts total**

**Some connection between nodes are ending in timeout**

```
  - alert: CassandraConnectionTimeoutsTotal
    expr: rate(cassandra_stats{name="org:apache:cassandra:metrics:connection:totaltimeouts:count"}[1m]) > 5
    for: 2m
    labels:
      severity: critical
    annotations:
      summary: Cassandra connection timeouts total (instance {{ $labels.instance }})
      description: "Some connection between nodes are ending in timeout\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


### **12. Cassandra storage exceptions**

**Something is going wrong with cassandra storage**

```
  - alert: CassandraStorageExceptions
    expr: changes(cassandra_stats{name="org:apache:cassandra:metrics:storage:exceptions:count"}[1m]) > 1
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Cassandra storage exceptions (instance {{ $labels.instance }})
      description: "Something is going wrong with cassandra storage\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **13. Cassandra tombstone dump**

**Too much tombstones scanned in queries**

```
  - alert: CassandraTombstoneDump
    expr: cassandra_stats{name="org:apache:cassandra:metrics:table:tombstonescannedhistogram:99thpercentile"} > 1000
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Cassandra tombstone dump (instance {{ $labels.instance }})
      description: "Too much tombstones scanned in queries\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **14. Cassandra client request unvailable write**

Write failures have occurred because too many nodes are unavailable

```
  - alert: CassandraClientRequestUnvailableWrite
    expr: changes(cassandra_stats{name="org:apache:cassandra:metrics:clientrequest:write:unavailables:count"}[1m]) > 0
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Cassandra client request unvailable write (instance {{ $labels.instance }})
      description: "Write failures have occurred because too many nodes are unavailable\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **15. Cassandra client request unvailable read**

Read failures have occurred because too many nodes are unavailable

```
  - alert: CassandraClientRequestUnvailableRead
    expr: changes(cassandra_stats{name="org:apache:cassandra:metrics:clientrequest:read:unavailables:count"}[1m]) > 0
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Cassandra client request unvailable read (instance {{ $labels.instance }})
      description: "Read failures have occurred because too many nodes are unavailable\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **16. Cassandra client request write failure**

A lot of write failures encountered. A write failure is a non-timeout exception encountered during a write request. Examine the reason map to find to the root cause. The most common cause for this type of error is when batch sizes are too large.

```
  - alert: CassandraClientRequestWriteFailure
    expr: increase(cassandra_stats{name="org:apache:cassandra:metrics:clientrequest:write:failures:oneminuterate"}[1m]) > 0
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Cassandra client request write failure (instance {{ $labels.instance }})
      description: "A lot of write failures encountered. A write failure is a non-timeout exception encountered during a write request. Examine the reason map to find to the root cause. The most common cause for this type of error is when batch sizes are too large.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **17. Cassandra client request read failure**

A lot of read failures encountered. A read failure is a non-timeout exception encountered during a read request. Examine the reason map to find to the root cause. The most common cause for this type of error is when batch sizes are too large.

```
  - alert: CassandraClientRequestReadFailure
    expr: increase(cassandra_stats{name="org:apache:cassandra:metrics:clientrequest:read:failures:oneminuterate"}[1m]) > 0
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Cassandra client request read failure (instance {{ $labels.instance }})
      description: "A lot of read failures encountered. A read failure is a non-timeout exception encountered during a read request. Examine the reason map to find to the root cause. The most common cause for this type of error is when batch sizes are too large.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **18. Cassandra cache hit rate key cache**

Key cache hit rate is below 85%

```
  - alert: CassandraCacheHitRateKeyCache
    expr: cassandra_stats{name="org:apache:cassandra:metrics:cache:keycache:hitrate:value"} < .85
    for: 2m
    labels:
      severity: critical
    annotations:
      summary: Cassandra cache hit rate key cache (instance {{ $labels.instance }})
      description: "Key cache hit rate is below 85%\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```