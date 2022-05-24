# 2 Java : java-client

[java-client](https://github.com/prometheus/client_java)

### **1. JVM memory filling up**

**JVM memory is filling up (> 80%)**

```
- alert: JvmMemoryFillingUp
  expr: jvm_memory_bytes_used / jvm_memory_bytes_max{area="heap"} > 0.8
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "JVM memory filling up (instance {{ $labels.instance }})"
    description: "JVM memory is filling up (> 80%)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```