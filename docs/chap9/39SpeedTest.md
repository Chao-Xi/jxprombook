# 2 SpeedTest : Speedtest exporter

[Speedtest exporter](https://github.com/nlamirault/speedtest_exporter)


### **1. SpeedTest Slow Internet Download**

**Internet download speed is currently `{{humanize $value}}` Mbps.**

```
  - alert: SpeedtestSlowInternetDownload
    expr: avg_over_time(speedtest_download[10m]) < 100
    for: 0m
    labels:
      severity: warning
    annotations:
      summary: SpeedTest Slow Internet Download (instance {{ $labels.instance }})
      description: "Internet download speed is currently {{humanize $value}} Mbps.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **2. SpeedTest Slow Internet Upload**

**Internet upload speed is currently `{{humanize $value}}` Mbps.**

```
  - alert: SpeedtestSlowInternetUpload
    expr: avg_over_time(speedtest_upload[10m]) < 20
    for: 0m
    labels:
      severity: warning
    annotations:
      summary: SpeedTest Slow Internet Upload (instance {{ $labels.instance }})
      description: "Internet upload speed is currently {{humanize $value}} Mbps.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

