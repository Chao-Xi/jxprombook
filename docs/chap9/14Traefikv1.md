# **14. Traefik**


## **1 Traefik : [Embedded exporter v2](https://docs.traefik.io/observability/metrics/prometheus/)**

### **1.1. Traefik service down**

**All Traefik services are down**

```
  - alert: TraefikServiceDown
    expr: count(traefik_service_server_up) by (service) == 0
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Traefik service down (instance {{ $labels.instance }})
      description: "All Traefik services are down\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


### **1.2. Traefik high HTTP 4xx error rate service**

**Traefik service 4xx error rate is above 5%**

```
  - alert: TraefikHighHttp4xxErrorRateService
    expr: sum(rate(traefik_service_requests_total{code=~"4.*"}[3m])) by (service) / sum(rate(traefik_service_requests_total[3m])) by (service) * 100 > 5
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: Traefik high HTTP 4xx error rate service (instance {{ $labels.instance }})
      description: "Traefik service 4xx error rate is above 5%\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **1.3. Traefik high HTTP 5xx error rate service**

**Traefik service 5xx error rate is above 5%**

```
  - alert: TraefikHighHttp5xxErrorRateService
    expr: sum(rate(traefik_service_requests_total{code=~"5.*"}[3m])) by (service) / sum(rate(traefik_service_requests_total[3m])) by (service) * 100 > 5
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: Traefik high HTTP 5xx error rate service (instance {{ $labels.instance }})
      description: "Traefik service 5xx error rate is above 5%\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


## **2. Traefik : [Embedded exporter v1](https://docs.traefik.io/observability/metrics/prometheus/)**

### **2.1. Traefik backend down**
 
**All Traefik backends are down**

```
- alert: TraefikBackendDown
  expr: count(traefik_backend_server_up) by (backend) == 0
  for: 5m
  labels:
    severity: error
  annotations:
    summary: "Traefik backend down (instance {{ $labels.instance }})"
    description: "All Traefik backends are down\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **2.2. Traefik backend errors**

**Traefik backend error rate is above 10%**

```
- alert: TraefikBackendErrors
  expr: sum(rate(traefik_backend_requests_total{code=~"5.*"}[5m])) by (backend) / sum(rate(traefik_backend_requests_total[5m])) by (backend) > 0.1
  for: 5m
  labels:
    severity: error
  annotations:
    summary: "Traefik backend errors (instance {{ $labels.instance }})"
    description: "Traefik backend error rate is above 10%\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **2.3. Traefik high HTTP 5xx error rate backend**

**Traefik backend 5xx error rate is above 5%**

```
  - alert: TraefikHighHttp5xxErrorRateBackend
    expr: sum(rate(traefik_backend_requests_total{code=~"5.*"}[3m])) by (backend) / sum(rate(traefik_backend_requests_total[3m])) by (backend) * 100 > 5
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: Traefik high HTTP 5xx error rate backend (instance {{ $labels.instance }})
      description: "Traefik backend 5xx error rate is above 5%\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```