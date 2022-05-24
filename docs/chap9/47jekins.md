# 5 Jenkins

https://plugins.jenkins.io/prometheus/

### **1. Jenkins offline**

**Jenkins offline: `{{$labels.instance}}` in realm `{{$labels.realm}}/{{$labels.env}}` (`{{$labels.region}}`)**

```
  - alert: JenkinsOffline
    expr: jenkins_node_offline_value > 1
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Jenkins offline (instance {{ $labels.instance }})
      description: "Jenkins offline: `{{$labels.instance}}` in realm {{$labels.realm}}/{{$labels.env}} ({{$labels.region}})\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **2. Jenkins healthcheck**

**Jenkins healthcheck score: `{{$value}}`. Healthcheck failure for `{{$labels.instance}}` in realm `{{$labels.realm}}/{{$labels.env}} `(`{{$labels.region}}`)**


```
  - alert: JenkinsHealthcheck
    expr: jenkins_health_check_score < 1
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Jenkins healthcheck (instance {{ $labels.instance }})
      description: "Jenkins healthcheck score: {{$value}}. Healthcheck failure for `{{$labels.instance}}` in realm {{$labels.realm}}/{{$labels.env}} ({{$labels.region}})\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **3. Jenkins outdated plugins**


**`{{ $value }}` plugins need update**

```
  - alert: JenkinsOutdatedPlugins
    expr: sum(jenkins_plugins_withUpdate) by (instance) > 3
    for: 1d
    labels:
      severity: warning
    annotations:
      summary: Jenkins outdated plugins (instance {{ $labels.instance }})
      description: "{{ $value }} plugins need update\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **4. Jenkins builds health score**

**Healthcheck failure for `{{$labels.instance}}` in realm `{{$labels.realm}}/{{$labels.env}}` (`{{$labels.region}}`)**

```
  - alert: JenkinsBuildsHealthScore
    expr: default_jenkins_builds_health_score < 1
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Jenkins builds health score (instance {{ $labels.instance }})
      description: "Healthcheck failure for `{{$labels.instance}}` in realm {{$labels.realm}}/{{$labels.env}} ({{$labels.region}})\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **5. Jenkins run failure total**

**Job run failures: (`{{$value}}`) `{{$labels.jenkins_job}}`. Healthcheck failure for `{{$labels.instance}}` in realm `{{$labels.realm}}/{{$labels.env}} ({{$labels.region}}`)**

```
  - alert: JenkinsRunFailureTotal
    expr: delta(jenkins_runs_failure_total[1h]) > 100
    for: 0m
    labels:
      severity: warning
    annotations:
      summary: Jenkins run failure total (instance {{ $labels.instance }})
      description: "Job run failures: ({{$value}}) {{$labels.jenkins_job}}. Healthcheck failure for `{{$labels.instance}}` in realm {{$labels.realm}}/{{$labels.env}} ({{$labels.region}})\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **6. Jenkins build tests failing**

**Last build tests failed: `{{$labels.jenkins_job}}`. Failed build Tests for job `{{$labels.jenkins_job}}` on `{{$labels.instance}}/{{$labels.env}}` (`{{$labels.region}}`)**

```
  - alert: JenkinsBuildTestsFailing
    expr: default_jenkins_builds_last_build_tests_failing > 0
    for: 0m
    labels:
      severity: warning
    annotations:
      summary: Jenkins build tests failing (instance {{ $labels.instance }})
      description: "Last build tests failed: {{$labels.jenkins_job}}. Failed build Tests for job `{{$labels.jenkins_job}}` on {{$labels.instance}}/{{$labels.env}} ({{$labels.region}})\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **7. Jenkins last build failed**

**Last build failed: `{{$labels.jenkins_job}}`. Failed build for job `{{$labels.jenkins_job}}` on `{{$labels.instance}}/{{$labels.env}} `(`{{$labels.region}}`)**

```
  # * RUNNING  -1 true  - The build had no errors.
  # * SUCCESS   0 true  - The build had no errors.
  # * UNSTABLE  1 true  - The build had some errors but they were not fatal. For example, some tests failed.
  # * FAILURE   2 false - The build had a fatal error.
  # * NOT_BUILT 3 false - The module was not built.
  # * ABORTED   4 false - The build was manually aborted.
  - alert: JenkinsLastBuildFailed
    expr: default_jenkins_builds_last_build_result_ordinal == 2
    for: 0m
    labels:
      severity: warning
    annotations:
      summary: Jenkins last build failed (instance {{ $labels.instance }})
      description: "Last build failed: {{$labels.jenkins_job}}. Failed build for job `{{$labels.jenkins_job}}` on {{$labels.instance}}/{{$labels.env}} ({{$labels.region}})\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

