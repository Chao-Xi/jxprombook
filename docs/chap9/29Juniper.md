# 7. Juniper : `junos_exporter`

[`czerwonk/junos_exporter`](https://github.com/czerwonk/junos_exporter)


### **1. Switch is down**

**The switch appears to be down**

```
- alert: SwitchIsDown
  expr: junos_up == 0
  for: 5m
  labels:
    severity: error
  annotations:
    summary: "Switch is down (instance {{ $labels.instance }})"
    description: "The switch appears to be down\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **2. High Bandwith Usage 1GiB**

**Interface is highly saturated for at least 1 min. (> 0.90GiB/s)**

```
- alert: HighBandwithUsage1gib
  expr: irate(junos_interface_transmit_bytes[1m]) * 8 > 1e+9 * 0.90
  for: 5m
  labels:
    severity: error
  annotations:
    summary: "High Bandwith Usage 1GiB (instance {{ $labels.instance }})"
    description: "Interface is highly saturated for at least 1 min. (> 0.90GiB/s)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **3. High Bandwith Usage 1GiB**

**Interface is getting saturated for at least 1 min. (> 0.80GiB/s)**

```
- alert: HighBandwithUsage1gib
  expr: irate(junos_interface_transmit_bytes[1m]) * 8 > 1e+9 * 0.80
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "High Bandwith Usage 1GiB (instance {{ $labels.instance }})"
    description: "Interface is getting saturated for at least 1 min. (> 0.80GiB/s)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```