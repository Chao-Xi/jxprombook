# 1. MySQL : prometheus/`mysqld_exporter`

[https://github.com/prometheus/mysqld_exporter](https://github.com/prometheus/mysqld_exporter)


* MYSQLDown
* MysqlQPSTooHigh
* [基于Prometheus构建MySQL可视化监控平台](https://github.com/Chao-Xi/JacobTechBlog/blob/master/k8s_dev/prometheus/36Prometheus_Mysql.md)

```
groups:
- name: MYSQL服务监控
  rules: 
  - alert: MYSQLDown
    expr: mysql_up != 1
    for: 10m
    labels:
      severity: critical
    annotations:
      summary: mysql server {{ $labels.realip }} is down. please check it in time.
  - alert: MysqlQPSTooHigh
    expr: (rate(mysql_global_status_queries{job="mysql_exporter"}[5m]) or irate(mysql_global_status_queries{job="mysql_exporter"}[5m])) > 300
    for: 10m
    labels:
      severity: critical
    annotations:
      summary: mysql server {{ $labels.realip }} QPS is too high. please keep an eyes on it.
```

### **1. MySQL down**

MySQL instance is down on `{{ $labels.instance }}`

```
  - alert: MysqlDown
    expr: mysql_up == 0
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: MySQL down (instance {{ $labels.instance }})
      description: "MySQL instance is down on {{ $labels.instance }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


### **1.2. MySQL too many connections (> 80%)**

**More than 80% of MySQL connections are in use on `{{ $labels.instance }}`**

```
  - alert: MysqlTooManyConnections(>80%)
    expr: max_over_time(mysql_global_status_threads_connected[1m]) / mysql_global_variables_max_connections * 100 > 80
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: MySQL too many connections (> 80%) (instance {{ $labels.instance }})
      description: "More than 80% of MySQL connections are in use on {{ $labels.instance }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **1.3. MySQL high threads running**

**More than 60% of MySQL connections are in running state on `{{ $labels.instance }}`**

```
  - alert: MysqlHighThreadsRunning
    expr: max_over_time(mysql_global_status_threads_running[1m]) / mysql_global_variables_max_connections * 100 > 60
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: MySQL high threads running (instance {{ $labels.instance }})
      description: "More than 60% of MySQL connections are in running state on {{ $labels.instance }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


### **1.4. MySQL Slave IO thread not running**

**MySQL Slave IO thread not running on `{{ $labels.instance }}`**

```
  - alert: MysqlSlaveIoThreadNotRunning
    expr: mysql_slave_status_master_server_id > 0 and ON (instance) mysql_slave_status_slave_io_running == 0
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: MySQL Slave IO thread not running (instance {{ $labels.instance }})
      description: "MySQL Slave IO thread not running on {{ $labels.instance }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


### **1.5. MySQL Slave SQL thread not running**

**MySQL Slave SQL thread not running on `{{ $labels.instance }}`**

```
  - alert: MysqlSlaveSqlThreadNotRunning
    expr: mysql_slave_status_master_server_id > 0 and ON (instance) mysql_slave_status_slave_sql_running == 0
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: MySQL Slave SQL thread not running (instance {{ $labels.instance }})
      description: "MySQL Slave SQL thread not running on {{ $labels.instance }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **1.6. MySQL Slave replication lag**

MySQL replication lag on `{{ $labels.instance }}`

```
  - alert: MysqlSlaveReplicationLag
    expr: mysql_slave_status_master_server_id > 0 and ON (instance) (mysql_slave_status_seconds_behind_master - mysql_slave_status_sql_delay) > 30
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: MySQL Slave replication lag (instance {{ $labels.instance }})
      description: "MySQL replication lag on {{ $labels.instance }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **1.7. MySQL slow queries**

**MySQL server mysql has some new slow query.**

```
  - alert: MysqlSlowQueries
    expr: increase(mysql_global_status_slow_queries[1m]) > 0
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: MySQL slow queries (instance {{ $labels.instance }})
      description: "MySQL server mysql has some new slow query.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **1.8. MySQL InnoDB log waits**

**MySQL innodb log writes stalling**

```
  - alert: MysqlInnodbLogWaits
    expr: rate(mysql_global_status_innodb_log_waits[15m]) > 10
    for: 0m
    labels:
      severity: warning
    annotations:
      summary: MySQL InnoDB log waits (instance {{ $labels.instance }})
      description: "MySQL innodb log writes stalling\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


### **1.9. MySQL restarted**

**MySQL has just been restarted, less than one minute ago on `{{ $labels.instance }}`.**

```
  - alert: MysqlRestarted
    expr: mysql_global_status_uptime < 60
    for: 0m
    labels:
      severity: info
    annotations:
      summary: MySQL restarted (instance {{ $labels.instance }})
      description: "MySQL has just been restarted, less than one minute ago on {{ $labels.instance }}.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

