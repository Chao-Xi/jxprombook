# 4 Cortex

### **1. Cortex ruler configuration reload failure**

**Cortex ruler configuration reload failure (instance `{{ $labels.instance }})`**

```
  - alert: CortexRulerConfigurationReloadFailure
    expr: cortex_ruler_config_last_reload_successful != 1
    for: 0m
    labels:
      severity: warning
    annotations:
      summary: Cortex ruler configuration reload failure (instance {{ $labels.instance }})
      description: "Cortex ruler configuration reload failure (instance {{ $labels.instance }})\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


### **2. Cortex not connected to Alertmanager**

**Cortex not connected to Alertmanager `(instance {{ $labels.instance }})`**

```
  - alert: CortexNotConnectedToAlertmanager
    expr: cortex_prometheus_notifications_alertmanagers_discovered < 1
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Cortex not connected to Alertmanager (instance {{ $labels.instance }})
      description: "Cortex not connected to Alertmanager (instance {{ $labels.instance }})\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **3. Cortex notification are being dropped**

**Cortex notification are being dropped due to errors (instance `{{ $labels.instance }}`)**

```
  - alert: CortexNotificationAreBeingDropped
    expr: rate(cortex_prometheus_notifications_dropped_total[5m]) > 0
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Cortex notification are being dropped (instance {{ $labels.instance }})
      description: "Cortex notification are being dropped due to errors (instance {{ $labels.instance }})\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **4. Cortex notification error**

**Cortex is failing when sending alert notifications (instance `{{ $labels.instance }}`)**

```
  - alert: CortexNotificationError
    expr: rate(cortex_prometheus_notifications_errors_total[5m]) > 0
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Cortex notification error (instance {{ $labels.instance }})
      description: "Cortex is failing when sending alert notifications (instance {{ $labels.instance }})\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **5. Cortex ingester unhealthy**

**Cortex has an unhealthy ingester**

```
  - alert: CortexIngesterUnhealthy
    expr: cortex_ring_members{state="Unhealthy", name="ingester"} > 0
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Cortex ingester unhealthy (instance {{ $labels.instance }})
      description: "Cortex has an unhealthy ingester\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **6. Cortex frontend queries stuck**

**There are queued up queries in query-frontend.**

```
  - alert: CortexFrontendQueriesStuck
    expr: sum by (job) (cortex_query_frontend_queue_length) > 0
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: Cortex frontend queries stuck (instance {{ $labels.instance }})
      description: "There are queued up queries in query-frontend.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```