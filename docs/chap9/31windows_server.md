# **5. Windows Server: `prometheus-community/windows_exporter`**

[https://github.com/prometheus-community/windows_exporter](https://github.com/prometheus-community/windows_exporter)

### **5.1. Windows Server collector Error**

**Collector `{{ $labels.collector }}` was not successful**

```
  - alert: WindowsServerCollectorError
    expr: windows_exporter_collector_success == 0
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Windows Server collector Error (instance {{ $labels.instance }})
      description: "Collector {{ $labels.collector }} was not successful\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **5.2 Windows Server service Status**

**Windows Service state is not OK**

```
  - alert: WindowsServerServiceStatus
    expr: windows_service_status{status="ok"} != 1
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: Windows Server service Status (instance {{ $labels.instance }})
      description: "Windows Service state is not OK\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **5.3. Windows Server CPU Usage**

**CPU Usage is more than 80%**

```
  - alert: WindowsServerCpuUsage
    expr: 100 - (avg by (instance) (rate(windows_cpu_time_total{mode="idle"}[2m])) * 100) > 80
    for: 0m
    labels:
      severity: warning
    annotations:
      summary: Windows Server CPU Usage (instance {{ $labels.instance }})
      description: "CPU Usage is more than 80%\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


### **5.4. Windows Server memory Usage**

**Memory usage is more than 90%**

```
  - alert: WindowsServerMemoryUsage
    expr: 100 - ((windows_os_physical_memory_free_bytes / windows_cs_physical_memory_bytes) * 100) > 90
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: Windows Server memory Usage (instance {{ $labels.instance }})
      description: "Memory usage is more than 90%\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **5.5. Windows Server disk Space Usage**

**Disk usage is more than 80%**

```
  - alert: WindowsServerDiskSpaceUsage
    expr: 100.0 - 100 * ((windows_logical_disk_free_bytes / 1024 / 1024 ) / (windows_logical_disk_size_bytes / 1024 / 1024)) > 80
    for: 2m
    labels:
      severity: critical
    annotations:
      summary: Windows Server disk Space Usage (instance {{ $labels.instance }})
      description: "Disk usage is more than 80%\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

