# 4. Blackbox : prometheus/blackbox_exporter

[`prometheus/blackbox_exporter`](https://github.com/prometheus/blackbox_exporter)

## `blackbox_exporter.yml`

```
modules:
  http_get_2xx:
    prober: http
    timeout: 10s
    http:
      valid_http_versions: ["HTTP/1.1", "HTTP/2"]
      method: GET
  baidu.com:
    prober: dns
    timeout: 10s
    dns:
      query_name: "baidu.com"
      query_type: "A"
      preferred_ip_protocol: "ip4"
```

### **26.1 Blackbox probe failed**

**Probe failed**

```
- alert: ProbeFailed
  expr: probe_success == 0
  for: 5m
  labels:
    severity: error
  annotations:
    summary: "Probe failed (instance {{ $labels.instance }})"
    description: "Probe failed\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **26.2. Blackbox slow probe**

**Blackbox probe took more than 1s to complete**

```
- alert: SlowProbe
  expr: avg_over_time(probe_duration_seconds[1m]) > 1
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "Slow probe (instance {{ $labels.instance }})"
    description: "Blackbox probe took more than 1s to complete\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **26.3. Blackbox probe HTTP failure**

**HTTP status code is not`200-399`**

```
- alert: HttpStatusCode
  expr: probe_http_status_code <= 199 OR probe_http_status_code >= 400
  for: 5m
  labels:
    severity: error
  annotations:
    summary: "HTTP Status Code (instance {{ $labels.instance }})"
    description: "HTTP status code is not 200-399\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```


### **26.4. SSL certificate will expire soon**

**SSL certificate expires in 30 days**

```
- alert: SslCertificateWillExpireSoon
  expr: probe_ssl_earliest_cert_expiry - time() < 86400 * 30
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "SSL certificate will expire soon (instance {{ $labels.instance }})"
    description: "SSL certificate expires in 30 days\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **26.5. SSL certificate expired**

**SSL certificate has expired already**

```
- alert: SslCertificateExpired
  expr: probe_ssl_earliest_cert_expiry - time()  <= 0
  for: 5m
  labels:
    severity: error
  annotations:
    summary: "SSL certificate expired (instance {{ $labels.instance }})"
    description: "SSL certificate has expired already\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **26.6. Blackbox probe slow HTTP**

**HTTP request took more than 1s**

```
- alert: HttpSlowRequests
  expr: avg_over_time(probe_http_duration_seconds[1m]) > 1
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "HTTP slow requests (instance {{ $labels.instance }})"
    description: "HTTP request took more than 1s\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **26.7. Blackbox probe slow ping**

**Blackbox ping took more than 1s**

```
- alert: SlowPing
  expr: avg_over_time(probe_icmp_duration_seconds[1m]) > 1
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "Slow ping (instance {{ $labels.instance }})"
    description: "Blackbox ping took more than 1s\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

