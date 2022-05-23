# **27. OpenEBS**

### **1. Used pool capacity**

**OpenEBS Pool use more than 80% of his capacity `\n VALUE = {{ $value }}\n LABELS: {{ $labels }}`**

```
- alert: UsedPoolCapacity
  expr: (openebs_used_pool_capacity_percent) > 80
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "Used pool capacity (instance {{ $labels.instance }})"
    description: "OpenEBS Pool use more than 80% of his capacity\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```


