# 3 Sidekiq

## **[Strech/sidekiq-prometheus-exporter](https://github.com/Strech/sidekiq-prometheus-exporter)**

### **1 Sidekiq queue size**

**Sidekiq queue `{{ $labels.name }}` is growing**

```
  - alert: SidekiqQueueSize
    expr: sidekiq_queue_size > 100
    for: 1m
    labels:
      severity: warning
    annotations:
      summary: Sidekiq queue size (instance {{ $labels.instance }})
      description: "Sidekiq queue {{ $labels.name }} is growing\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **2 Sidekiq scheduling latency too high**

**Sidekiq jobs are taking more than 1min to be picked up. Users may be seeing delays in background processing.**

```
  - alert: SidekiqSchedulingLatencyTooHigh
    expr: max(sidekiq_queue_latency) > 60
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Sidekiq scheduling latency too high (instance {{ $labels.instance }})
      description: "Sidekiq jobs are taking more than 1min to be picked up. Users may be seeing delays in background processing.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```