# **13. HaProxy `haproxy_exporter`**



## **1 [Embedded exporter (HAProxy >= v2)](https://github.com/haproxy/haproxy/tree/master/contrib/prometheus-exporter)**

### **1.1.HAProxy high HTTP 4xx error rate backend**

**Too many HTTP requests with status 4xx (> 5%) on backend `{{ $labels.fqdn }}/{{ $labels.backend }}`**

```
  - alert: HaproxyHighHttp4xxErrorRateBackend
    expr: ((sum by (proxy) (rate(haproxy_server_http_responses_total{code="4xx"}[1m])) / sum by (proxy) (rate(haproxy_server_http_responses_total[1m]))) * 100) > 5
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: HAProxy high HTTP 4xx error rate backend (instance {{ $labels.instance }})
      description: "Too many HTTP requests with status 4xx (> 5%) on backend {{ $labels.fqdn }}/{{ $labels.backend }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **1.2. HAProxy high HTTP 5xx error rate backend**

**Too many HTTP requests with status 5xx (> 5%) on backend `{{ $labels.fqdn }}/{{ $labels.backend }}`**

```
  - alert: HaproxyHighHttp5xxErrorRateBackend
    expr: ((sum by (proxy) (rate(haproxy_server_http_responses_total{code="5xx"}[1m])) / sum by (proxy) (rate(haproxy_server_http_responses_total[1m]))) * 100) > 5
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: HAProxy high HTTP 5xx error rate backend (instance {{ $labels.instance }})
      description: "Too many HTTP requests with status 5xx (> 5%) on backend {{ $labels.fqdn }}/{{ $labels.backend }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


### **1.3. HAProxy high HTTP 4xx error rate server**

**Too many HTTP requests with status 4xx (> 5%) on server `{{ $labels.server }}`**

```
  - alert: HaproxyHighHttp4xxErrorRateServer
    expr: ((sum by (server) (rate(haproxy_server_http_responses_total{code="4xx"}[1m])) / sum by (server) (rate(haproxy_server_http_responses_total[1m]))) * 100) > 5
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: HAProxy high HTTP 4xx error rate server (instance {{ $labels.instance }})
      description: "Too many HTTP requests with status 4xx (> 5%) on server {{ $labels.server }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **1.4. HAProxy high HTTP 5xx error rate server**

**Too many HTTP requests with status 5xx (> 5%) on server `{{ $labels.server }}`**

```
  - alert: HaproxyHighHttp5xxErrorRateServer
    expr: ((sum by (server) (rate(haproxy_server_http_responses_total{code="5xx"}[1m])) / sum by (server) (rate(haproxy_server_http_responses_total[1m]))) * 100) > 5
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: HAProxy high HTTP 5xx error rate server (instance {{ $labels.instance }})
      description: "Too many HTTP requests with status 5xx (> 5%) on server {{ $labels.server }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


### **1.5. HAProxy server response errors**

**Too many response errors to `{{ $labels.server }}` server (> 5%).**

```
  - alert: HaproxyServerResponseErrors
    expr: (sum by (server) (rate(haproxy_server_response_errors_total[1m])) / sum by (server) (rate(haproxy_server_http_responses_total[1m]))) * 100 > 5
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: HAProxy server response errors (instance {{ $labels.instance }})
      description: "Too many response errors to {{ $labels.server }} server (> 5%).\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **1.6. HAProxy backend connection errors**

**Too many connection errors to `{{ $labels.fqdn }}/{{ $labels.backend }}` backend (> 100 req/s). Request throughput may be too high.**

```
  - alert: HaproxyBackendConnectionErrors
    expr: (sum by (proxy) (rate(haproxy_backend_connection_errors_total[1m]))) > 100
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: HAProxy backend connection errors (instance {{ $labels.instance }})
      description: "Too many connection errors to {{ $labels.fqdn }}/{{ $labels.backend }} backend (> 100 req/s). Request throughput may be too high.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **1.7. HAProxy server connection errors**

Too many connection errors to `{{ $labels.server }}` server (> 100 req/s). Request throughput may be too high.

```
  - alert: HaproxyServerConnectionErrors
    expr: (sum by (proxy) (rate(haproxy_server_connection_errors_total[1m]))) > 100
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: HAProxy server connection errors (instance {{ $labels.instance }})
      description: "Too many connection errors to {{ $labels.server }} server (> 100 req/s). Request throughput may be too high.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


### **1.8. HAProxy backend max active session > 80%**

**Session limit from backend `{{ $labels.proxy }}` to server `{{ $labels.server }}` reached 80% of limit - `{{ $value | printf "%.2f"}}%`**

```
  - alert: HaproxyBackendMaxActiveSession>80%
    expr: ((haproxy_server_max_sessions >0) * 100) / (haproxy_server_limit_sessions > 0) > 80
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: HAProxy backend max active session > 80% (instance {{ $labels.instance }})
      description: "Session limit from backend {{ $labels.proxy }} to server {{ $labels.server }} reached 80% of limit - {{ $value | printf \"%.2f\"}}%\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **1.9. HAProxy pending requests**

Some HAProxy requests are pending on `{{ $labels.proxy }} - {{ $value | printf "%.2f"}}`

```
  - alert: HaproxyPendingRequests
    expr: sum by (proxy) (rate(haproxy_backend_current_queue[2m])) > 0
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: HAProxy pending requests (instance {{ $labels.instance }})
      description: "Some HAProxy requests are pending on {{ $labels.proxy }} - {{ $value | printf \"%.2f\"}}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **1.10. HAProxy HTTP slowing down**

Average request time is increasing - `{{ $value | printf "%.2f"}}`

```
  - alert: HaproxyHttpSlowingDown
    expr: avg by (proxy) (haproxy_backend_max_total_time_seconds) > 1
    for: 1m
    labels:
      severity: warning
    annotations:
      summary: HAProxy HTTP slowing down (instance {{ $labels.instance }})
      description: "Average request time is increasing - {{ $value | printf \"%.2f\"}}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **1.11. HAProxy retry high**

**High rate of retry on `{{ $labels.proxy }} - {{ $value | printf "%.2f"}}`**

```
  - alert: HaproxyRetryHigh
    expr: sum by (proxy) (rate(haproxy_backend_retry_warnings_total[1m])) > 10
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: HAProxy retry high (instance {{ $labels.instance }})
      description: "High rate of retry on {{ $labels.proxy }} - {{ $value | printf \"%.2f\"}}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


### **1.12. HAproxy has no alive backends**

**HAProxy has no alive active or backup backends for `{{ $labels.proxy }}`**

```
  - alert: HaproxyHasNoAliveBackends
    expr: haproxy_backend_active_servers + haproxy_backend_backup_servers == 0
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: HAproxy has no alive backends (instance {{ $labels.instance }})
      description: "HAProxy has no alive active or backup backends for {{ $labels.proxy }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


### **1.13. HAProxy frontend security blocked requests**

**HAProxy is blocking requests for security reason**

```
  - alert: HaproxyFrontendSecurityBlockedRequests
    expr: sum by (proxy) (rate(haproxy_frontend_denied_connections_total[2m])) > 10
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: HAProxy frontend security blocked requests (instance {{ $labels.instance }})
      description: "HAProxy is blocking requests for security reason\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **1.14. HAProxy server healthcheck failure**

**Some server healthcheck are failing on `{{ $labels.server }}`**

```
  - alert: HaproxyServerHealthcheckFailure
    expr: increase(haproxy_server_check_failures_total[1m]) > 0
    for: 1m
    labels:
      severity: warning
    annotations:
      summary: HAProxy server healthcheck failure (instance {{ $labels.instance }})
      description: "Some server healthcheck are failing on {{ $labels.server }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


## **2 HaProxy : [prometheus/haproxy_exporter (HAProxy < v2)](https://github.com/prometheus/haproxy_exporter)**


### **2.1. HAProxy down**

**HAProxy down**

```
  - alert: HaproxyDown
    expr: haproxy_up == 0
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: HAProxy down (instance {{ $labels.instance }})
      description: "HAProxy down\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


### **2.2. HAProxy high HTTP 4xx error rate backend**

**Too many HTTP requests with status 4xx (> 5%) on backend `{{ $labels.fqdn }}/{{ $labels.backend }}`**

```
  - alert: HaproxyHighHttp4xxErrorRateBackend
    expr: sum by (backend) (rate(haproxy_server_http_responses_total{code="4xx"}[1m])) / sum by (backend) (rate(haproxy_server_http_responses_total[1m]) * 100) > 5
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: HAProxy high HTTP 4xx error rate backend (instance {{ $labels.instance }})
      description: "Too many HTTP requests with status 4xx (> 5%) on backend {{ $labels.fqdn }}/{{ $labels.backend }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **2.3. HAProxy high HTTP 5xx error rate backend**

**Too many HTTP requests with status 5xx (> 5%) on backend `{{ $labels.fqdn }}/{{ $labels.backend }}`**

```
  - alert: HaproxyHighHttp5xxErrorRateBackend
    expr: sum by (backend) (rate(haproxy_server_http_responses_total{code="5xx"}[1m])) / sum by (backend) (rate(haproxy_server_http_responses_total[1m]) * 100) > 5
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: HAProxy high HTTP 5xx error rate backend (instance {{ $labels.instance }})
      description: "Too many HTTP requests with status 5xx (> 5%) on backend {{ $labels.fqdn }}/{{ $labels.backend }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **2.4. HAProxy high HTTP 4xx error rate server**

**Too many HTTP requests with status 4xx (> 5%) on server `{{ $labels.server }}`**

```
  - alert: HaproxyHighHttp4xxErrorRateServer
    expr: sum by (server) (rate(haproxy_server_http_responses_total{code="4xx"}[1m])) / sum by (server) (rate(haproxy_server_http_responses_total[1m]) * 100) > 5
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: HAProxy high HTTP 4xx error rate server (instance {{ $labels.instance }})
      description: "Too many HTTP requests with status 4xx (> 5%) on server {{ $labels.server }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **2.5. HAProxy high HTTP 5xx error rate server**

**Too many HTTP requests with status 5xx (> 5%) on server `{{ $labels.server }}`**

```
  - alert: HaproxyHighHttp5xxErrorRateServer
    expr: sum by (server) (rate(haproxy_server_http_responses_total{code="5xx"}[1m])) / sum by (server) (rate(haproxy_server_http_responses_total[1m]) * 100) > 5
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: HAProxy high HTTP 5xx error rate server (instance {{ $labels.instance }})
      description: "Too many HTTP requests with status 5xx (> 5%) on server {{ $labels.server }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **2.6. HAProxy server response errors**

**Too many response errors to `{{ $labels.server }}` server (> 5%).**

```
  - alert: HaproxyServerResponseErrors
    expr: sum by (server) (rate(haproxy_server_response_errors_total[1m])) / sum by (server) (rate(haproxy_server_http_responses_total[1m]) * 100) > 5
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: HAProxy server response errors (instance {{ $labels.instance }})
      description: "Too many response errors to {{ $labels.server }} server (> 5%).\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


### **2.7. HAProxy backend connection errors**

**Too many connection errors to `{{ $labels.fqdn }}/{{ $labels.backend }} `backend (> 100 req/s). Request throughput may be too high.**

```
  - alert: HaproxyBackendConnectionErrors
    expr: sum by (backend) (rate(haproxy_backend_connection_errors_total[1m])) > 100
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: HAProxy backend connection errors (instance {{ $labels.instance }})
      description: "Too many connection errors to {{ $labels.fqdn }}/{{ $labels.backend }} backend (> 100 req/s). Request throughput may be too high.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


### **2.8. HAProxy server connection errors**

**Too many connection errors to `{{ $labels.server }}` server (> 100 req/s). Request throughput may be too high.**

```
  - alert: HaproxyServerConnectionErrors
    expr: sum by (server) (rate(haproxy_server_connection_errors_total[1m])) > 100
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: HAProxy server connection errors (instance {{ $labels.instance }})
      description: "Too many connection errors to {{ $labels.server }} server (> 100 req/s). Request throughput may be too high.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


### **2.9. HAProxy backend max active session**

**HAproxy backend `{{ $labels.fqdn }}/{{ $labels.backend }}` is reaching session limit (> 80%).**

```
  - alert: HaproxyBackendMaxActiveSession
    expr: ((sum by (backend) (avg_over_time(haproxy_backend_max_sessions[2m])) / sum by (backend) (avg_over_time(haproxy_backend_limit_sessions[2m]))) * 100) > 80
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: HAProxy backend max active session (instance {{ $labels.instance }})
      description: "HAproxy backend {{ $labels.fqdn }}/{{ $labels.backend }} is reaching session limit (> 80%).\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **2.10. HAProxy pending requests**

**Some HAProxy requests are pending on `{{ $labels.fqdn }}/{{ $labels.backend }}` backend**

```
  - alert: HaproxyPendingRequests
    expr: sum by (backend) (haproxy_backend_current_queue) > 0
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: HAProxy pending requests (instance {{ $labels.instance }})
      description: "Some HAProxy requests are pending on {{ $labels.fqdn }}/{{ $labels.backend }} backend\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


### **2.11. HAProxy HTTP slowing down**

**Average request time is increasing**

```
  - alert: HaproxyHttpSlowingDown
    expr: avg by (backend) (haproxy_backend_http_total_time_average_seconds) > 1
    for: 1m
    labels:
      severity: warning
    annotations:
      summary: HAProxy HTTP slowing down (instance {{ $labels.instance }})
      description: "Average request time is increasing\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


### **2.12. HAProxy retry high**

**High rate of retry on `{{ $labels.fqdn }}/{{ $labels.backend }}` backend**

```
  - alert: HaproxyRetryHigh
    expr: sum by (backend) (rate(haproxy_backend_retry_warnings_total[1m])) > 10
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: HAProxy retry high (instance {{ $labels.instance }})
      description: "High rate of retry on {{ $labels.fqdn }}/{{ $labels.backend }} backend\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


### **2.13. HAProxy backend down**

**HAProxy backend is down**

```
  - alert: HaproxyBackendDown
    expr: haproxy_backend_up == 0
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: HAProxy backend down (instance {{ $labels.instance }})
      description: "HAProxy backend is down\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **2.14. HAProxy server down**

**HAProxy server is down**

```
  - alert: HaproxyServerDown
    expr: haproxy_server_up == 0
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: HAProxy server down (instance {{ $labels.instance }})
      description: "HAProxy server is down\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


### **2.15. HAProxy frontend security blocked requests**

**HAProxy is blocking requests for security reason**

```
  - alert: HaproxyFrontendSecurityBlockedRequests
    expr: sum by (frontend) (rate(haproxy_frontend_requests_denied_total[2m])) > 10
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: HAProxy frontend security blocked requests (instance {{ $labels.instance }})
      description: "HAProxy is blocking requests for security reason\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


### **2.16. HAProxy server healthcheck failure**

**Some server healthcheck are failing on `{{ $labels.server }}`**

```
  - alert: HaproxyServerHealthcheckFailure
    expr: increase(haproxy_server_check_failures_total[1m]) > 0
    for: 1m
    labels:
      severity: warning
    annotations:
      summary: HAProxy server healthcheck failure (instance {{ $labels.instance }})
      description: "Some server healthcheck are failing on {{ $labels.server }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


