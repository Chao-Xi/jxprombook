# **12 Solr : embedded exporter** 

[embedded exporter ](https://solr.apache.org/guide/8_11/monitoring-solr-with-prometheus-and-grafana.html)

### **1. Solr update errors**

Solr collection `{{ $labels.collection }}`has failed updates for replica `{{ $labels.replica }} on {{ $labels.base_url }}`.

```
  - alert: SolrUpdateErrors
    expr: increase(solr_metrics_core_update_handler_errors_total[1m]) > 1
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Solr update errors (instance {{ $labels.instance }})
      description: "Solr collection {{ $labels.collection }} has failed updates for replica {{ $labels.replica }} on {{ $labels.base_url }}.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **2. Solr query errors**

Solr has increased query errors in collection `{{ $labels.collection }}` for replica `{{ $labels.replica }}` on `{{ $labels.base_url }}`.

```
  - alert: SolrQueryErrors
    expr: increase(solr_metrics_core_errors_total{category="QUERY"}[1m]) > 1
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: Solr query errors (instance {{ $labels.instance }})
      description: "Solr has increased query errors in collection {{ $labels.collection }} for replica {{ $labels.replica }} on {{ $labels.base_url }}.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **3. Solr replication errors**

Solr collection `{{ $labels.collection }}` has failed updates for replica `{{ $labels.replica }}` on `{{ $labels.base_url }}`.

```
  - alert: SolrReplicationErrors
    expr: increase(solr_metrics_core_errors_total{category="REPLICATION"}[1m]) > 1
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Solr replication errors (instance {{ $labels.instance }})
      description: "Solr collection {{ $labels.collection }} has failed updates for replica {{ $labels.replica }} on {{ $labels.base_url }}.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


### **4. Solr low live node count**

Solr collection `{{ $labels.collection }}` has less than two live nodes for replica `{{ $labels.replica }}` on `{{ $labels.base_url }}`.

```
  - alert: SolrLowLiveNodeCount
    expr: solr_collections_live_nodes < 2
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Solr low live node count (instance {{ $labels.instance }})
      description: "Solr collection {{ $labels.collection }} has less than two live nodes for replica {{ $labels.replica }} on {{ $labels.base_url }}.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```