# **19. Nomad : prometheus-nomad-exporter**

[samber/prometheus-nomad-exporter](https://github.com/samber/prometheus-nomad-exporter)

### **1. Nomad job failed**

**Nomad job failed**

```
  - alert: NomadJobFailed
    expr: nomad_nomad_job_summary_failed > 0
    for: 0m
    labels:
      severity: warning
    annotations:
      summary: Nomad job failed (instance {{ $labels.instance }})
      description: "Nomad job failed\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **2. Nomad job lost**

**Nomad job lost**

```
  - alert: NomadJobLost
    expr: nomad_nomad_job_summary_lost > 0
    for: 0m
    labels:
      severity: warning
    annotations:
      summary: Nomad job lost (instance {{ $labels.instance }})
      description: "Nomad job lost\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **3. Nomad job queued**

**Nomad job queued**

```
  - alert: NomadJobQueued
    expr: nomad_nomad_job_summary_queued > 0
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: Nomad job queued (instance {{ $labels.instance }})
      description: "Nomad job queued\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **4. Nomad blocked evaluation**

**Nomad blocked evaluation**

```
  - alert: NomadBlockedEvaluation
    expr: nomad_nomad_blocked_evals_total_blocked > 0
    for: 0m
    labels:
      severity: warning
    annotations:
      summary: Nomad blocked evaluation (instance {{ $labels.instance }})
      description: "Nomad blocked evaluation\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```