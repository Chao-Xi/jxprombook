# 6. SSL/TLS : ssl_exporter

### **1. SSL certificate probe failed**

**Failed to fetch SSL information `{{ $labels.instance }}`**

```
  - alert: SslCertificateProbeFailed
    expr: ssl_probe_success == 0
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: SSL certificate probe failed (instance {{ $labels.instance }})
      description: "Failed to fetch SSL information {{ $labels.instance }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **2. SSL certificate OSCP status unknown**

**Failed to get the OSCP status `{{ $labels.instance }}`**

```
  - alert: SslCertificateOscpStatusUnknown
    expr: ssl_ocsp_response_status == 2
    for: 0m
    labels:
      severity: warning
    annotations:
      summary: SSL certificate OSCP status unknown (instance {{ $labels.instance }})
      description: "Failed to get the OSCP status {{ $labels.instance }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **3. SSL certificate revoked**

**SSL certificate revoked `{{ $labels.instance }}`**

```
  - alert: SslCertificateRevoked
    expr: ssl_ocsp_response_status == 1
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: SSL certificate revoked (instance {{ $labels.instance }})
      description: "SSL certificate revoked {{ $labels.instance }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **4. SSL certificate expiry (< 7 days)**

**`{{ $labels.instance }}` Certificate is expiring in 7 days**

```
  - alert: SslCertificateExpiry(<7Days)
    expr: ssl_verified_cert_not_after{chain_no="0"} - time() < 86400 * 7
    for: 0m
    labels:
      severity: warning
    annotations:
      summary: SSL certificate expiry (< 7 days) (instance {{ $labels.instance }})
      description: "{{ $labels.instance }} Certificate is expiring in 7 days\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

