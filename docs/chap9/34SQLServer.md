# 3 SQL Server

[Ozarklake/prometheus-mssql-exporter](https://github.com/Ozarklake/prometheus-mssql-exporter)


### **1. SQL Server down**

**SQL server instance is down**

```
  - alert: SqlServerDown
    expr: mssql_up == 0
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: SQL Server down (instance {{ $labels.instance }})
      description: "SQL server instance is down\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **8.2. SQL Server deadlock**

**SQL Server is having some deadlock.**

```
  - alert: SqlServerDeadlock
    expr: increase(mssql_deadlocks[1m]) > 5
    for: 0m
    labels:
      severity: warning
    annotations:
      summary: SQL Server deadlock (instance {{ $labels.instance }})
      description: "SQL Server is having some deadlock.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

