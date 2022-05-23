# **7 Netdata : Embedded exporter**

[Embedded exporter](https://github.com/netdata/netdata/blob/c7f2647a62f73b5bfec7545c019b11efadbca2bb/exporting/prometheus/README.md)

### **7.1. Netdata high cpu usage**

**Netdata high CPU usage (> 80%)**

```
  - alert: NetdataHighCpuUsage
    expr: rate(netdata_cpu_cpu_percentage_average{dimension="idle"}[1m]) > 80
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: Netdata high cpu usage (instance {{ $labels.instance }})
      description: "Netdata high CPU usage (> 80%)\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


### **7.2. Host CPU steal noisy neighbor**

**CPU steal is > 10%. A noisy neighbor is killing VM performances or a spot instance may be out of credit.**

```
  - alert: HostCpuStealNoisyNeighbor
    expr: rate(netdata_cpu_cpu_percentage_average{dimension="steal"}[1m]) > 10
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: Host CPU steal noisy neighbor (instance {{ $labels.instance }})
      description: "CPU steal is > 10%. A noisy neighbor is killing VM performances or a spot instance may be out of credit.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **7.3. Netdata high memory usage**

**Netdata high memory usage (> 80%)**

```
  - alert: NetdataHighMemoryUsage
    expr: 100 / netdata_system_ram_MB_average * netdata_system_ram_MB_average{dimension=~"free|cached"} < 20
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: Netdata high memory usage (instance {{ $labels.instance }})
      description: "Netdata high memory usage (> 80%)\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **7.4. Netdata low disk space**

**Netdata low disk space (> 80%)**

```
  - alert: NetdataLowDiskSpace
    expr: 100 / netdata_disk_space_GB_average * netdata_disk_space_GB_average{dimension=~"avail|cached"} < 20
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: Netdata low disk space (instance {{ $labels.instance }})
      description: "Netdata low disk space (> 80%)\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **7.5. Netdata predicted disk full**

**Netdata predicted disk full in 24 hours**

```
  - alert: NetdataPredictedDiskFull
    expr: predict_linear(netdata_disk_space_GB_average{dimension=~"avail|cached"}[3h], 24 * 3600) < 0
    for: 0m
    labels:
      severity: warning
    annotations:
      summary: Netdata predicted disk full (instance {{ $labels.instance }})
      description: "Netdata predicted disk full in 24 hours\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **7.6. Netdata MD mismatch cnt unsynchronized blocks**

**RAID Array have unsynchronized blocks**

```
  - alert: NetdataMdMismatchCntUnsynchronizedBlocks
    expr: netdata_md_mismatch_cnt_unsynchronized_blocks_average > 1024
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: Netdata MD mismatch cnt unsynchronized blocks (instance {{ $labels.instance }})
      description: "RAID Array have unsynchronized blocks\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **7.7. Netdata disk reallocated sectors**

**Reallocated sectors on disk**

```
  - alert: NetdataDiskReallocatedSectors
    expr: increase(netdata_smartd_log_reallocated_sectors_count_sectors_average[1m]) > 0
    for: 0m
    labels:
      severity: info
    annotations:
      summary: Netdata disk reallocated sectors (instance {{ $labels.instance }})
      description: "Reallocated sectors on disk\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **7.8. Netdata disk current pending sector**

**Disk current pending sector**

```
  - alert: NetdataDiskCurrentPendingSector
    expr: netdata_smartd_log_current_pending_sector_count_sectors_average > 0
    for: 0m
    labels:
      severity: warning
    annotations:
      summary: Netdata disk current pending sector (instance {{ $labels.instance }})
      description: "Disk current pending sector\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **7.9. Netdata reported uncorrectable disk sectors**

**Reported uncorrectable disk sectors**

```
  - alert: NetdataReportedUncorrectableDiskSectors
    expr: increase(netdata_smartd_log_offline_uncorrectable_sector_count_sectors_average[2m]) > 0
    for: 0m
    labels:
      severity: warning
    annotations:
      summary: Netdata reported uncorrectable disk sectors (instance {{ $labels.instance }})
      description: "Reported uncorrectable disk sectors\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```