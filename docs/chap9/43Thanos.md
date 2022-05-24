# 1 Thanos

### **1. Thanos compaction halted**

**Thanos compaction has failed to run and is now halted.**

```
  - alert: ThanosCompactionHalted
    expr: thanos_compact_halted == 1
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Thanos compaction halted (instance {{ $labels.instance }})
      description: "Thanos compaction has failed to run and is now halted.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


### **2. Thanos compact bucket operation failure**

**Thanos compaction has failing storage operations**


```
  - alert: ThanosCompactBucketOperationFailure
    expr: rate(thanos_objstore_bucket_operation_failures_total[1m]) > 0
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Thanos compact bucket operation failure (instance {{ $labels.instance }})
      description: "Thanos compaction has failing storage operations\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **3. Thanos compact not run**

**Thanos compaction has not run in 24 hours.**

```
  - alert: ThanosCompactNotRun
    expr: (time() - thanos_objstore_bucket_last_successful_upload_time) > 24*60*60
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Thanos compact not run (instance {{ $labels.instance }})
      description: "Thanos compaction has not run in 24 hours.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```