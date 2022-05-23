# **18. Kubernetes : kube-state-metrics**

[kube-state-metrics](https://github.com/kubernetes/kube-state-metrics/tree/master/docs)


### **1. Kubernetes Node ready**

Node `{{ $labels.node }}` has been unready for a long time

```
  - alert: KubernetesNodeReady
    expr: kube_node_status_condition{condition="Ready",status="true"} == 0
    for: 10m
    labels:
      severity: critical
    annotations:
      summary: Kubernetes Node ready (instance {{ $labels.instance }})
      description: "Node {{ $labels.node }} has been unready for a long time\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```



### **2. Kubernetes MemoryPressure**


**`{{ $labels.node }}` has MemoryPressure condition**

```
- alert: KubernetesMemorypressure
  expr: kube_node_status_condition{condition="MemoryPressure",status="true"} == 1
  for: 5m
  labels:
    severity: error
  annotations:
    summary: "Kubernetes MemoryPressure (instance {{ $labels.instance }})"
    description: "{{ $labels.node }} has MemoryPressure condition\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **3. Kubernetes DiskPressure**

**`{{ $labels.node }}` has DiskPressure condition**

```
- alert: KubernetesDiskpressure
  expr: kube_node_status_condition{condition="DiskPressure",status="true"} == 1
  for: 5m
  labels:
    severity: error
  annotations:
    summary: "Kubernetes DiskPressure (instance {{ $labels.instance }})"
    description: "{{ $labels.node }} has DiskPressure condition\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **4. Kubernetes OutOfDisk**

**` {{ $labels.node }}` has OutOfDisk condition**

```
- alert: KubernetesOutofdisk
  expr: kube_node_status_condition{condition="OutOfDisk",status="true"} == 1
  for: 5m
  labels:
    severity: error
  annotations:
    summary: "Kubernetes OutOfDisk (instance {{ $labels.instance }})"
    description: "{{ $labels.node }} has OutOfDisk condition\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **5. Kubernetes out of capacity**

**`{{ $labels.node }}` is out of capacity**

```
  - alert: KubernetesOutOfCapacity
    expr: sum by (node) ((kube_pod_status_phase{phase="Running"} == 1) + on(uid) group_left(node) (0 * kube_pod_info{pod_template_hash=""})) / sum by (node) (kube_node_status_allocatable{resource="pods"}) * 100 > 90
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: Kubernetes out of capacity (instance {{ $labels.instance }})
      description: "{{ $labels.node }} is out of capacity\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **6. Kubernetes container oom killer**

**Container `{{ $labels.container }}` in pod `{{ $labels.namespace }}/{{ $labels.pod }}` has been OOMKilled `{{ $value }}` times in the last 10 minutes.**

```
  - alert: KubernetesContainerOomKiller
    expr: (kube_pod_container_status_restarts_total - kube_pod_container_status_restarts_total offset 10m >= 1) and ignoring (reason) min_over_time(kube_pod_container_status_last_terminated_reason{reason="OOMKilled"}[10m]) == 1
    for: 0m
    labels:
      severity: warning
    annotations:
      summary: Kubernetes container oom killer (instance {{ $labels.instance }})
      description: "Container {{ $labels.container }} in pod {{ $labels.namespace }}/{{ $labels.pod }} has been OOMKilled {{ $value }} times in the last 10 minutes.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **7. Kubernetes Job failed**

**Job `{{$labels.namespace}}/{{$labels.exported_job}} ` failed to complete**

```
- alert: KubernetesJobFailed
  expr: kube_job_status_failed > 0
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "Kubernetes Job failed (instance {{ $labels.instance }})"
    description: "Job {{$labels.namespace}}/{{$labels.exported_job}} failed to complete\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **8. Kubernetes CronJob suspended**

**CronJob `{{ $labels.namespace }}/{{ $labels.cronjob }}` is suspended**

```
- alert: KubernetesCronjobSuspended
  expr: kube_cronjob_spec_suspend != 0
  for: 5m
  labels:
    severity: info
  annotations:
    summary: "Kubernetes CronJob suspended (instance {{ $labels.instance }})"
    description: "CronJob {{ $labels.namespace }}/{{ $labels.cronjob }} is suspended\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **9. Kubernetes PersistentVolumeClaim pending**

**PersistentVolumeClaim `{{ $labels.namespace }}/{{ $labels.persistentvolumeclaim }} ` is pending**

```
- alert: KubernetesPersistentvolumeclaimPending
  expr: kube_persistentvolumeclaim_status_phase{phase="Pending"} == 1
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "Kubernetes PersistentVolumeClaim pending (instance {{ $labels.instance }})"
    description: "PersistentVolumeClaim {{ $labels.namespace }}/{{ $labels.persistentvolumeclaim }} is pending\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **10. Volume out of disk space**

**Volume is almost full (< 10% left)**

```
- alert: VolumeOutOfDiskSpace
  expr: kubelet_volume_stats_available_bytes / kubelet_volume_stats_capacity_bytes * 100 < 10
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "Volume out of disk space (instance {{ $labels.instance }})"
    description: "Volume is almost full (< 10% left)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```
### **11. Volume full in four days**

**`{{ $labels.namespace }}/{{ $labels.persistentvolumeclaim }}` is expected to fill up within four days. Currently `{{ $value | humanize }}%` is available.**

```
- alert: VolumeFullInFourDays
  expr: 100 * (kubelet_volume_stats_available_bytes / kubelet_volume_stats_capacity_bytes) < 15 and predict_linear(kubelet_volume_stats_available_bytes[6h], 4 * 24 * 3600) < 0
  for: 5m
  labels:
    severity: error
  annotations:
    summary: "Volume full in four days (instance {{ $labels.instance }})"
    description: "{{ $labels.namespace }}/{{ $labels.persistentvolumeclaim }} is expected to fill up within four days. Currently {{ $value | humanize }}% is available.\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **12. Kubernetes PersistentVolume error**

**Persistent volume is in bad state**

```
  - alert: KubernetesPersistentvolumeError
    expr: kube_persistentvolume_status_phase{phase=~"Failed|Pending", job="kube-state-metrics"} > 0
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Kubernetes PersistentVolume error (instance {{ $labels.instance }})
      description: "Persistent volume is in bad state\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **13. StatefulSet down**

**A StatefulSet went down**

```
- alert: StatefulsetDown
  expr: (kube_statefulset_status_replicas_ready / kube_statefulset_status_replicas_current) != 1
  for: 5m
  labels:
    severity: error
  annotations:
    summary: "StatefulSet down (instance {{ $labels.instance }})"
    description: "A StatefulSet went down\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

### **14. Kubernetes HPA scaling ability**

**Pod is unable to scale**

```
  - alert: KubernetesHpaScalingAbility
    expr: kube_horizontalpodautoscaler_status_condition{status="false", condition="AbleToScale"} == 1
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: Kubernetes HPA scaling ability (instance {{ $labels.instance }})
      description: "Pod is unable to scale\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **15. Kubernetes HPA metric availability**

**HPA is not able to collect metrics**

```
  - alert: KubernetesHpaMetricAvailability
    expr: kube_horizontalpodautoscaler_status_condition{status="false", condition="ScalingActive"} == 1
    for: 0m
    labels:
      severity: warning
    annotations:
      summary: Kubernetes HPA metric availability (instance {{ $labels.instance }})
      description: "HPA is not able to collect metrics\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **16. Kubernetes HPA scale capability**

**The maximum number of desired Pods has been hit**

```
  - alert: KubernetesHpaScaleCapability
    expr: kube_horizontalpodautoscaler_status_desired_replicas >= kube_horizontalpodautoscaler_spec_max_replicas
    for: 2m
    labels:
      severity: info
    annotations:
      summary: Kubernetes HPA scale capability (instance {{ $labels.instance }})
      description: "The maximum number of desired Pods has been hit\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **17. Kubernetes Pod not healthy**

**Pod has been in a non-ready state for longer than 15 minutes.**

```
  - alert: KubernetesPodNotHealthy
    expr: min_over_time(sum by (namespace, pod) (kube_pod_status_phase{phase=~"Pending|Unknown|Failed"})[15m:1m]) > 0
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Kubernetes Pod not healthy (instance {{ $labels.instance }})
      description: "Pod has been in a non-ready state for longer than 15 minutes.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **18. Kubernetes pod crash looping**

**Pod `{{ $labels.pod }}` is crash looping**

```
  - alert: KubernetesPodCrashLooping
    expr: increase(kube_pod_container_status_restarts_total[1m]) > 3
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: Kubernetes pod crash looping (instance {{ $labels.instance }})
      description: "Pod {{ $labels.pod }} is crash looping\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **19. Kubernetes ReplicasSet mismatch**

**Deployment Replicas mismatch**

```
  - alert: KubernetesReplicassetMismatch
    expr: kube_replicaset_spec_replicas != kube_replicaset_status_ready_replicas
    for: 10m
    labels:
      severity: warning
    annotations:
      summary: Kubernetes ReplicasSet mismatch (instance {{ $labels.instance }})
      description: "Deployment Replicas mismatch\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **20. Kubernetes Deployment replicas mismatch**


**Deployment Replicas mismatch**

```
  - alert: KubernetesDeploymentReplicasMismatch
    expr: kube_deployment_spec_replicas != kube_deployment_status_replicas_available
    for: 10m
    labels:
      severity: warning
    annotations:
      summary: Kubernetes Deployment replicas mismatch (instance {{ $labels.instance }})
      description: "Deployment Replicas mismatch\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **21. Kubernetes StatefulSet replicas mismatch**

**A StatefulSet does not match the expected number of replicas.**

```
  - alert: KubernetesStatefulsetReplicasMismatch
    expr: kube_statefulset_status_replicas_ready != kube_statefulset_status_replicas
    for: 10m
    labels:
      severity: warning
    annotations:
      summary: Kubernetes StatefulSet replicas mismatch (instance {{ $labels.instance }})
      description: "A StatefulSet does not match the expected number of replicas.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **22. Kubernetes Deployment generation mismatch**

**A Deployment has failed but has not been rolled back.**

```
  - alert: KubernetesDeploymentGenerationMismatch
    expr: kube_deployment_status_observed_generation != kube_deployment_metadata_generation
    for: 10m
    labels:
      severity: critical
    annotations:
      summary: Kubernetes Deployment generation mismatch (instance {{ $labels.instance }})
      description: "A Deployment has failed but has not been rolled back.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


### **23. Kubernetes StatefulSet generation mismatch**

**A StatefulSet has failed but has not been rolled back.**


```
  - alert: KubernetesStatefulsetGenerationMismatch
    expr: kube_statefulset_status_observed_generation != kube_statefulset_metadata_generation
    for: 10m
    labels:
      severity: critical
    annotations:
      summary: Kubernetes StatefulSet generation mismatch (instance {{ $labels.instance }})
      description: "A StatefulSet has failed but has not been rolled back.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **24. Kubernetes StatefulSet update not rolled out**

**StatefulSet update has not been rolled out.**

```
  - alert: KubernetesStatefulsetUpdateNotRolledOut
    expr: max without (revision) (kube_statefulset_status_current_revision unless kube_statefulset_status_update_revision) * (kube_statefulset_replicas != kube_statefulset_status_replicas_updated)
    for: 10m
    labels:
      severity: warning
    annotations:
      summary: Kubernetes StatefulSet update not rolled out (instance {{ $labels.instance }})
      description: "StatefulSet update has not been rolled out.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **25. Kubernetes DaemonSet rollout stuck**

**Some Pods of DaemonSet are not scheduled or not ready**

```
  - alert: KubernetesDaemonsetRolloutStuck
    expr: kube_daemonset_status_number_ready / kube_daemonset_status_desired_number_scheduled * 100 < 100 or kube_daemonset_status_desired_number_scheduled - kube_daemonset_status_current_number_scheduled > 0
    for: 10m
    labels:
      severity: warning
    annotations:
      summary: Kubernetes DaemonSet rollout stuck (instance {{ $labels.instance }})
      description: "Some Pods of DaemonSet are not scheduled or not ready\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **26. Kubernetes DaemonSet misscheduled**

**Some DaemonSet Pods are running where they are not supposed to run**

```
  - alert: KubernetesDaemonsetMisscheduled
    expr: kube_daemonset_status_number_misscheduled > 0
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: Kubernetes DaemonSet misscheduled (instance {{ $labels.instance }})
      description: "Some DaemonSet Pods are running where they are not supposed to run\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **27. Kubernetes CronJob too long**

CronJob `{{ $labels.namespace }}/{{ $labels.cronjob }}` is taking more than 1h to complete.

```
  - alert: KubernetesCronjobTooLong
    expr: time() - kube_cronjob_next_schedule_time > 3600
    for: 0m
    labels:
      severity: warning
    annotations:
      summary: Kubernetes CronJob too long (instance {{ $labels.instance }})
      description: "CronJob {{ $labels.namespace }}/{{ $labels.cronjob }} is taking more than 1h to complete.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **28. Kubernetes job slow completion**

Kubernetes Job `{{ $labels.namespace }}/{{ $labels.job_name }}` did not complete in time.

```
  - alert: KubernetesJobSlowCompletion
    expr: kube_job_spec_completions - kube_job_status_succeeded > 0
    for: 12h
    labels:
      severity: critical
    annotations:
      summary: Kubernetes job slow completion (instance {{ $labels.instance }})
      description: "Kubernetes Job {{ $labels.namespace }}/{{ $labels.job_name }} did not complete in time.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **29. Kubernetes API server errors**

**Kubernetes API server is experiencing high error rate**

```
  - alert: KubernetesApiServerErrors
    expr: sum(rate(apiserver_request_total{job="apiserver",code=~"^(?:5..)$"}[1m])) / sum(rate(apiserver_request_total{job="apiserver"}[1m])) * 100 > 3
    for: 2m
    labels:
      severity: critical
    annotations:
      summary: Kubernetes API server errors (instance {{ $labels.instance }})
      description: "Kubernetes API server is experiencing high error rate\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **30. Kubernetes API client errors**

**Kubernetes API client is experiencing high error rate**

```
  - alert: KubernetesApiClientErrors
    expr: (sum(rate(rest_client_requests_total{code=~"(4|5).."}[1m])) by (instance, job) / sum(rate(rest_client_requests_total[1m])) by (instance, job)) * 100 > 1
    for: 2m
    labels:
      severity: critical
    annotations:
      summary: Kubernetes API client errors (instance {{ $labels.instance }})
      description: "Kubernetes API client is experiencing high error rate\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **31. Kubernetes client certificate expires next week**

**A client certificate used to authenticate to the apiserver is expiring next week.**

```
  - alert: KubernetesClientCertificateExpiresNextWeek
    expr: apiserver_client_certificate_expiration_seconds_count{job="apiserver"} > 0 and histogram_quantile(0.01, sum by (job, le) (rate(apiserver_client_certificate_expiration_seconds_bucket{job="apiserver"}[5m]))) < 7*24*60*60
    for: 0m
    labels:
      severity: warning
    annotations:
      summary: Kubernetes client certificate expires next week (instance {{ $labels.instance }})
      description: "A client certificate used to authenticate to the apiserver is expiring next week.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **32. Kubernetes client certificate expires soon**

**A client certificate used to authenticate to the apiserver is expiring in less than 24.0 hours.**

```
  - alert: KubernetesClientCertificateExpiresSoon
    expr: apiserver_client_certificate_expiration_seconds_count{job="apiserver"} > 0 and histogram_quantile(0.01, sum by (job, le) (rate(apiserver_client_certificate_expiration_seconds_bucket{job="apiserver"}[5m]))) < 24*60*60
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Kubernetes client certificate expires soon (instance {{ $labels.instance }})
      description: "A client certificate used to authenticate to the apiserver is expiring in less than 24.0 hours.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **33. Kubernetes API server latency**

**Kubernetes API server has a 99th percentile latency of `{{ $value }}` seconds for `{{ $labels.verb }} {{ $labels.resource }}`.**

```
  - alert: KubernetesApiServerLatency
    expr: histogram_quantile(0.99, sum(rate(apiserver_request_latencies_bucket{subresource!="log",verb!~"^(?:CONNECT|WATCHLIST|WATCH|PROXY)$"} [10m])) WITHOUT (instance, resource)) / 1e+06 > 1
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: Kubernetes API server latency (instance {{ $labels.instance }})
      description: "Kubernetes API server has a 99th percentile latency of {{ $value }} seconds for {{ $labels.verb }} {{ $labels.resource }}.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


