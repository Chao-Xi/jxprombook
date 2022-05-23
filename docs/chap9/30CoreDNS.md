# **30. CoreDNS**

### **1. CoreDNS Panic Count**

**Number of CoreDNS panics encountered**

```
- alert: CorednsPanicCount
  expr: increase(coredns_panic_count_total[10m]) > 0
  for: 5m
  labels:
    severity: error
  annotations:
    summary: "CoreDNS Panic Count (instance {{ $labels.instance }})"
    description: "Number of CoreDNS panics encountered\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

