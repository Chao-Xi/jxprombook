# **9 Freeswitch**

[znerol/prometheus-freeswitch-exporter](https://pypi.org/project/prometheus-freeswitch-exporter)

### **1. Freeswitch down**

**Freeswitch is unresponsive**

```
  - alert: FreeswitchDown
    expr: freeswitch_up == 0
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Freeswitch down (instance {{ $labels.instance }})
      description: "Freeswitch is unresponsive\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **2. Freeswitch Sessions Warning**

**High sessions usage on `{{ $labels.instance }}: {{ $value | printf "%.2f"}}%`**

```
  - alert: FreeswitchSessionsWarning
    expr: (freeswitch_session_active * 100 / freeswitch_session_limit) > 80
    for: 10m
    labels:
      severity: warning
    annotations:
      summary: Freeswitch Sessions Warning (instance {{ $labels.instance }})
      description: "High sessions usage on {{ $labels.instance }}: {{ $value | printf \"%.2f\"}}%\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **3. Freeswitch Sessions Critical**

**High sessions usage on `{{ $labels.instance }}: {{ $value | printf "%.2f"}}%`**

```
  - alert: FreeswitchSessionsCritical
    expr: (freeswitch_session_active * 100 / freeswitch_session_limit) > 90
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: Freeswitch Sessions Critical (instance {{ $labels.instance }})
      description: "High sessions usage on {{ $labels.instance }}: {{ $value | printf \"%.2f\"}}%\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

