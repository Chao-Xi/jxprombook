# **9 PGBouncer**

[spreaker/prometheus-pgbouncer-exporter ](https://github.com/spreaker/prometheus-pgbouncer-exporter)

###  **9.1 PGBouncer active connections**

**PGBouncer pools are filling up**

```
  - alert: PgbouncerActiveConnections
    expr: pgbouncer_pools_server_active_connections > 200
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: PGBouncer active connections (instance {{ $labels.instance }})
      description: "PGBouncer pools are filling up\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **9.2. PGBouncer errors**

**PGBouncer is logging errors. This may be due to a a server restart or an admin typing commands at the pgbouncer console.**

```
  - alert: PgbouncerErrors
    expr: increase(pgbouncer_errors_count{errmsg!="server conn crashed?"}[1m]) > 10
    for: 0m
    labels:
      severity: warning
    annotations:
      summary: PGBouncer errors (instance {{ $labels.instance }})
      description: "PGBouncer is logging errors. This may be due to a a server restart or an admin typing commands at the pgbouncer console.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **9.3. PGBouncer max connections**

**The number of PGBouncer client connections has reached `max_client_conn`.**

```
  - alert: PgbouncerMaxConnections
    expr: increase(pgbouncer_errors_count{errmsg="no more connections allowed (max_client_conn)"}[30s]) > 0
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: PGBouncer max connections (instance {{ $labels.instance }})
      description: "The number of PGBouncer client connections has reached max_client_conn.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```