# 21. Etcd

### **21.1. Insufficient Members**

**Etcd cluster should have an odd number of members**

```
- alert: InsufficientMembers
  expr: count(etcd_server_id) > (count(etcd_server_id) / 2 - 1)
  for: 5m
  labels:
    severity: error
  annotations:
    summary: "Insufficient Members (instance {{ $labels.instance }})"
    description: "Etcd cluster should have an odd number of members\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **21.2. No Leader**

**Etcd cluster have no leader**

```
- alert: NoLeader
  expr: etcd_server_has_leader == 0
  for: 5m
  labels:
    severity: error
  annotations:
    summary: "No Leader (instance {{ $labels.instance }})"
    description: "Etcd cluster have no leader\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **21.3. High number of leader changes**

**Etcd leader changed more than 3 times during last hour**

```
- alert: HighNumberOfLeaderChanges
  expr: increase(etcd_server_leader_changes_seen_total[1h]) > 3
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "High number of leader changes (instance {{ $labels.instance }})"
    description: "Etcd leader changed more than 3 times during last hour\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **21.4. High number of failed GRPC requests**

**More than 1% GRPC request failure detected in Etcd for 5 minutes**

```
- alert: HighNumberOfFailedGrpcRequests
  expr: sum(rate(grpc_server_handled_total{grpc_code!="OK"}[5m])) BY (grpc_service, grpc_method) / sum(rate(grpc_server_handled_total[5m])) BY (grpc_service, grpc_method) > 0.01
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "High number of failed GRPC requests (instance {{ $labels.instance }})"
    description: "More than 1% GRPC request failure detected in Etcd for 5 minutes\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **21.5. High number of failed GRPC requests**

**More than 5% GRPC request failure detected in Etcd for 5 minutes**

```
- alert: HighNumberOfFailedGrpcRequests
  expr: sum(rate(grpc_server_handled_total{grpc_code!="OK"}[5m])) BY (grpc_service, grpc_method) / sum(rate(grpc_server_handled_total[5m])) BY (grpc_service, grpc_method) > 0.05
  for: 5m
  labels:
    severity: error
  annotations:
    summary: "High number of failed GRPC requests (instance {{ $labels.instance }})"
    description: "More than 5% GRPC request failure detected in Etcd for 5 minutes\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **21.6. GRPC requests slow**

**GRPC requests slowing down, 99th percentil is over 0.15s for 5 minutes**

```
- alert: GrpcRequestsSlow
  expr: histogram_quantile(0.99, sum(rate(grpc_server_handling_seconds_bucket{grpc_type="unary"}[5m])) by (grpc_service, grpc_method, le)) > 0.15
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "GRPC requests slow (instance {{ $labels.instance }})"
    description: "GRPC requests slowing down, 99th percentil is over 0.15s for 5 minutes\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **21.7. High number of failed HTTP requests**

**More than 1% HTTP failure detected in Etcd for 5 minutes**

```
- alert: HighNumberOfFailedHttpRequests
  expr: sum(rate(etcd_http_failed_total[5m])) BY (method) / sum(rate(etcd_http_received_total[5m])) BY (method) > 0.01
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "High number of failed HTTP requests (instance {{ $labels.instance }})"
    description: "More than 1% HTTP failure detected in Etcd for 5 minutes\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

## **21.8. High number of failed HTTP requests**

**More than 5% HTTP failure detected in Etcd for 5 minutes**

```
- alert: HighNumberOfFailedHttpRequests
  expr: sum(rate(etcd_http_failed_total[5m])) BY (method) / sum(rate(etcd_http_received_total[5m])) BY (method) > 0.05
  for: 5m
  labels:
    severity: error
  annotations:
    summary: "High number of failed HTTP requests (instance {{ $labels.instance }})"
    description: "More than 5% HTTP failure detected in Etcd for 5 minutes\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **21.9. HTTP requests slow**

**HTTP requests slowing down, 99th percentil is over 0.15s for 5 minutes**

```
- alert: HttpRequestsSlow
  expr: histogram_quantile(0.99, rate(etcd_http_successful_duration_seconds_bucket[5m])) > 0.15
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "HTTP requests slow (instance {{ $labels.instance }})"
    description: "HTTP requests slowing down, 99th percentil is over 0.15s for 5 minutes\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **21.10. Etcd member communication slow**

**Etcd member communication slowing down, 99th percentil is over 0.15s for 5 minutes**

```
- alert: EtcdMemberCommunicationSlow
  expr: histogram_quantile(0.99, rate(etcd_network_peer_round_trip_time_seconds_bucket[5m])) > 0.15
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "Etcd member communication slow (instance {{ $labels.instance }})"
    description: "Etcd member communication slowing down, 99th percentil is over 0.15s for 5 minutes\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **21.11. High number of failed proposals**

**Etcd server got more than 5 failed proposals past hour**

```
- alert: HighNumberOfFailedProposals
  expr: increase(etcd_server_proposals_failed_total[1h]) > 5
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "High number of failed proposals (instance {{ $labels.instance }})"
    description: "Etcd server got more than 5 failed proposals past hour\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **21.12. High fsync durations**


**Etcd WAL fsync duration increasing, 99th percentil is over 0.5s for 5 minutes**


```
- alert: HighFsyncDurations
  expr: histogram_quantile(0.99, rate(etcd_disk_wal_fsync_duration_seconds_bucket[5m])) > 0.5
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "High fsync durations (instance {{ $labels.instance }})"
    description: "Etcd WAL fsync duration increasing, 99th percentil is over 0.5s for 5 minutes\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **21.13. High commit durations**


**Etcd commit duration increasing, 99th percentil is over 0.25s for 5 minutes**

```
- alert: HighCommitDurations
  expr: histogram_quantile(0.99, rate(etcd_disk_backend_commit_duration_seconds_bucket[5m])) > 0.25
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "High commit durations (instance {{ $labels.instance }})"
    description: "Etcd commit duration increasing, 99th percentil is over 0.25s for 5 minutes\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```



