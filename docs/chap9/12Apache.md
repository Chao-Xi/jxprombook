# 5 Apache `apache_exporter`

[Lusitaniae/apache_exporter](https://github.com/Lusitaniae/apache_exporter)

### **1. Apache down**

**Apache down**


```
  - alert: ApacheDown
    expr: apache_up == 0
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Apache down (instance {{ $labels.instance }})
      description: "Apache down\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **2. Apache workers load**

**Apache workers in busy state approach the max workers count 80% workers busy on `{{ $labels.instance }}`**


```
  - alert: ApacheWorkersLoad
    expr: (sum by (instance) (apache_workers{state="busy"}) / sum by (instance) (apache_scoreboard) ) * 100 > 80
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: Apache workers load (instance {{ $labels.instance }})
      description: "Apache workers in busy state approach the max workers count 80% workers busy on {{ $labels.instance }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


### **3. Apache restart**

**Apache has just been restarted.**

```
  - alert: ApacheRestart
    expr: apache_uptime_seconds_total / 60 < 1
    for: 0m
    labels:
      severity: warning
    annotations:
      summary: Apache restart (instance {{ $labels.instance }})
      description: "Apache has just been restarted.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

