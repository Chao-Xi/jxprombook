# 2. PostgreSQL

[https://github.com/wrouesnel/postgres_exporter/](https://github.com/wrouesnel/postgres_exporter/)

### **1. PostgreSQL down**

**PostgreSQL instance is down**

```
- alert: PostgresqlDown
  expr: pg_up == 0
  for: 5m
  labels:
    severity: error
  annotations:
    summary: "PostgreSQL down (instance {{ $labels.instance }})"
    description: "PostgreSQL instance is down\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **2. Postgresql restarted**

**Postgresql restarted**

```
  - alert: PostgresqlRestarted
    expr: time() - pg_postmaster_start_time_seconds < 60
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Postgresql restarted (instance {{ $labels.instance }})
      description: "Postgresql restarted\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```




### **3. Postgresql exporter error**

**Postgresql exporter is showing errors. A query may be buggy in query.yaml**

```
  - alert: PostgresqlExporterError
    expr: pg_exporter_last_scrape_error > 0
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Postgresql exporter error (instance {{ $labels.instance }})
      description: "Postgresql exporter is showing errors. A query may be buggy in query.yaml\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **4. Replication lag**

**PostgreSQL replication lag is going up (> 10s)**

```
- alert: ReplicationLag
  expr: pg_replication_lag > 10
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "Replication lag (instance {{ $labels.instance }})"
    description: "PostgreSQL replication lag is going up (> 10s)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **5 Table not vaccumed**

**Table has not been vaccum for 24 hours**

```
- alert: TableNotVaccumed
  expr: time() - pg_stat_user_tables_last_autovacuum > 60 * 60 * 24
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "Table not vaccumed (instance {{ $labels.instance }})"
    description: "Table has not been vaccum for 24 hours\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```


### **6. Table not analyzed**

**Table has not been analyzed for 24 hours**
 
```
- alert: TableNotAnalyzed
  expr: time() - pg_stat_user_tables_last_autoanalyze > 60 * 60 * 24
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "Table not analyzed (instance {{ $labels.instance }})"
    description: "Table has not been analyzed for 24 hours\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **7. Too many connections**

**PostgreSQL instance has too many connections**

```
- alert: TooManyConnections
  expr: sum by (datname) (pg_stat_activity_count{datname!~"template.*|postgres"}) > 100
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "Too many connections (instance {{ $labels.instance }})"
    description: "PostgreSQL instance has too many connections\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```


### **8. Not enough connections**

**PostgreSQL instance should have more connections (> 5)**
 
```
- alert: NotEnoughConnections
  expr: sum by (datname) (pg_stat_activity_count{datname!~"template.*|postgres"}) < 5
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "Not enough connections (instance {{ $labels.instance }})"
    description: "PostgreSQL instance should have more connections (> 5)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **9. Dead locks**

**PostgreSQL has dead-locks**

```
- alert: DeadLocks
  expr: rate(pg_stat_database_deadlocks{datname!~"template.*|postgres"}[1m]) > 0
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "Dead locks (instance {{ $labels.instance }})"
    description: "PostgreSQL has dead-locks\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **10. High rollback rate**

**Ratio of transactions being aborted compared to committed is > 2 %**

```
- alert: HighRollbackRate
  expr: rate(pg_stat_database_xact_rollback{datname!~"template.*"}[3m]) / rate(pg_stat_database_xact_commit{datname!~"template.*"}[3m]) > 0.02
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "High rollback rate (instance {{ $labels.instance }})"
    description: "Ratio of transactions being aborted compared to committed is > 2 %\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **11. Postgresql commit rate low**

**Postgresql seems to be processing very few transactions**

```
  - alert: PostgresqlCommitRateLow
    expr: rate(pg_stat_database_xact_commit[1m]) < 10
    for: 2m
    labels:
      severity: critical
    annotations:
      summary: Postgresql commit rate low (instance {{ $labels.instance }})
      description: "Postgresql seems to be processing very few transactions\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **12. Postgresql low XID consumption**

**Postgresql seems to be consuming transaction IDs very slowly**

```
  - alert: PostgresqlLowXidConsumption
    expr: rate(pg_txid_current[1m]) < 5
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: Postgresql low XID consumption (instance {{ $labels.instance }})
      description: "Postgresql seems to be consuming transaction IDs very slowly\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **13. Postgresql high rate statement timeout**

**Postgres transactions showing high rate of statement timeouts**

```
  - alert: PostgresqlHighRateStatementTimeout
    expr: rate(postgresql_errors_total{type="statement_timeout"}[1m]) > 3
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Postgresql high rate statement timeout (instance {{ $labels.instance }})
      description: "Postgres transactions showing high rate of statement timeouts\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **14. Postgresql high rate deadlock**

**Postgres detected deadlocks**

```
  - alert: PostgresqlHighRateDeadlock
    expr: increase(postgresql_errors_total{type="deadlock_detected"}[1m]) > 1
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Postgresql high rate deadlock (instance {{ $labels.instance }})
      description: "Postgres detected deadlocks\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **15. Postgresql unused replication slot**

**Unused Replication Slots**

```
  - alert: PostgresqlUnusedReplicationSlot
    expr: pg_replication_slots_active == 0
    for: 1m
    labels:
      severity: warning
    annotations:
      summary: Postgresql unused replication slot (instance {{ $labels.instance }})
      description: "Unused Replication Slots\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

```

### **16. Postgresql too many dead tuples**

**PostgreSQL dead tuples is too large**

```
  - alert: PostgresqlTooManyDeadTuples
    expr: ((pg_stat_user_tables_n_dead_tup > 10000) / (pg_stat_user_tables_n_live_tup + pg_stat_user_tables_n_dead_tup)) >= 0.1 unless ON(instance) (pg_replication_is_replica == 1)
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: Postgresql too many dead tuples (instance {{ $labels.instance }})
      description: "PostgreSQL dead tuples is too large\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **17. Postgresql split brain**

**Split Brain, too many primary Postgresql databases in read-write mode**

```
  - alert: PostgresqlSplitBrain
    expr: count(pg_replication_is_replica == 0) != 1
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Postgresql split brain (instance {{ $labels.instance }})
      description: "Split Brain, too many primary Postgresql databases in read-write mode\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

```

### **18. Postgresql promoted node**

**Postgresql standby server has been promoted as primary node**

```
  - alert: PostgresqlPromotedNode
    expr: pg_replication_is_replica and changes(pg_replication_is_replica[1m]) > 0
    for: 0m
    labels:
      severity: warning
    annotations:
      summary: Postgresql promoted node (instance {{ $labels.instance }})
      description: "Postgresql standby server has been promoted as primary node\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **19. Postgresql configuration changed**

**Postgres Database configuration change has occurred**

```
  - alert: PostgresqlConfigurationChanged
    expr: {__name__=~"pg_settings_.*"} != ON(__name__) {__name__=~"pg_settings_([^t]|t[^r]|tr[^a]|tra[^n]|tran[^s]|trans[^a]|transa[^c]|transac[^t]|transact[^i]|transacti[^o]|transactio[^n]|transaction[^_]|transaction_[^r]|transaction_r[^e]|transaction_re[^a]|transaction_rea[^d]|transaction_read[^_]|transaction_read_[^o]|transaction_read_o[^n]|transaction_read_on[^l]|transaction_read_onl[^y]).*"} OFFSET 5m
    for: 0m
    labels:
      severity: info
    annotations:
      summary: Postgresql configuration changed (instance {{ $labels.instance }})
      description: "Postgres Database configuration change has occurred\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **20. Postgresql SSL compression active**

Database connections with SSL compression enabled. This may add significant jitter in replication delay. Replicas should turn off SSL compression via `sslcompression=0` in `recovery.conf`.

```
  - alert: PostgresqlSslCompressionActive
    expr: sum(pg_stat_ssl_compression) > 0
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Postgresql SSL compression active (instance {{ $labels.instance }})
      description: "Database connections with SSL compression enabled. This may add significant jitter in replication delay. Replicas should turn off SSL compression via `sslcompression=0` in `recovery.conf`.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


### **21. Postgresql too many locks acquired**

Too many locks acquired on the database. If this alert happens frequently, we may need to increase the postgres setting `max_locks_per_transaction`.

```
  - alert: PostgresqlTooManyLocksAcquired
    expr: ((sum (pg_locks_count)) / (pg_settings_max_locks_per_transaction * pg_settings_max_connections)) > 0.20
    for: 2m
    labels:
      severity: critical
    annotations:
      summary: Postgresql too many locks acquired (instance {{ $labels.instance }})
      description: "Too many locks acquired on the database. If this alert happens frequently, we may need to increase the postgres setting max_locks_per_transaction.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

```

### **22. Slow queries**

**PostgreSQL executes slow queries (> 1min)**

```
- alert: SlowQueries
  expr: avg(rate(pg_stat_activity_max_tx_duration{datname!~"template.*"}[1m])) BY (datname) > 60
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "Slow queries (instance {{ $labels.instance }})"
    description: "PostgreSQL executes slow queries (> 1min)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```


