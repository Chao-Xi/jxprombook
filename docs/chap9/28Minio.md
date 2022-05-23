# **28. Minio**

### **1. Disk down**

**Minio Disk is down\n `VALUE = {{ $value }}\n LABELS: {{ $labels }}`**

```
- alert: DiskDown
  expr: minio_offline_disks > 0
  for: 5m
  labels:
    severity: error
  annotations:
    summary: "Disk down (instance {{ $labels.instance }})"
    description: "Minio Disk is down\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **2.Minio disk space usage**

**Minio available free space is low (< 10%)**

```
  - alert: MinioDiskSpaceUsage
    expr: disk_storage_available / disk_storage_total * 100 < 10
    for: 0m
    labels:
      severity: warning
    annotations:
      summary: Minio disk space usage (instance {{ $labels.instance }})
      description: "Minio available free space is low (< 10%)\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```
