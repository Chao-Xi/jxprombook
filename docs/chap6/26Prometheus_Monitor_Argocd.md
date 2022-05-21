# **Chap2 Prometheus Operator Monitor on ArgoCD**

1. 第一步，为`Argocd`创建自定义的报警规则
2. 第二步建立一个 `ServiceMonitor` 对象，用于 `Prometheus` 添加监控项
3. 第三步为 `ServiceMonitor` 对象关联 `metrics` 数据接口的一个 `Service` 对象
4. 第四步确保 `Service` 对象可以正确获取到 `metrics` 数据

### **argocd已经暴露metrics**

* `argocd-metrics`: 8082
*  `argocd-server-metrics`: 8083 

```
$ kubectl get svc -n argocd --show-labels | grep metrics
argocd-metrics          ClusterIP   100.70.123.135   <none>        8082/TCP            116d   app.kubernetes.io/component=metrics,app.kubernetes.io/name=argocd-metrics,app.kubernetes.io/part-of=argocd
argocd-server-metrics   ClusterIP   100.70.2.220     <none>        8083/TCP            116d   app.kubernetes.io/component=server,app.kubernetes.io/name=argocd-server-metrics,app.kubernetes.io/part-of=argocd
```

## **创建 ServiceMonitor** 


### **`servicemonitors/argocd-metrics.yaml`**

```
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: argocd-metrics
  namespace: argocd
  labels:
    prometheus: kube-prometheus
spec:
  selector:
    matchLabels:
      "app.kubernetes.io/name": argocd-metrics
  namespaceSelector:
    matchNames:
    - argocd
  jobLabel: app.kubernetes.io/name
  endpoints:
  - port: metrics
    interval: 10s
```

* 上面我们在 `argocd ` 命名空间下面创建了名为 `argocd-metrics` 的 `ServiceMonitor` 对象，基本属性和前面章节中的一致，
* 匹配 `argocd` 这个命名空间下面的具有 `"app.kubernetes.io/name": argocd-metrics` 这个 `label` 标签的 `Service`，`jobLabel` 表示用于检索 `job` 任务名称的标签，

```
$ kubectl create -f argocd-metrics.yaml
```

### **`servicemonitors/argocd-repo-server-metrics.yaml`**

```
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: argocd-repo-server-metrics
  namespace: argocd
  labels:
    prometheus: kube-prometheus
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: argocd-repo-server
  namespaceSelector:
    matchNames:
    - argocd
  jobLabel: app.kubernetes.io/name
  endpoints:
  - port: metrics
```

* 上面我们在 `argocd ` 命名空间下面创建了名为 `argocd-repo-server-metrics` 的 `ServiceMonitor` 对象，基本属性和前面章节中的一致，
* 匹配 `argocd` 这个命名空间下面的具有 `"app.kubernetes.io/name": argocd-repo-server` 这个 `label` 标签的 `Service`，`jobLabel` 表示用于检索 `job` 任务名称的标签，

### **`argocd-server-metrics.yaml`**

```
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: argocd-server-metrics
  namespace: argocd
  labels:
    prometheus: kube-prometheus
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: argocd-server-metrics
  namespaceSelector:
    matchNames:
    - argocd
  endpoints:
  - port: metrics
```

* 上面我们在 `argocd ` 命名空间下面创建了名为 `argocd-server-metrics` 的 `ServiceMonitor` 对象，基本属性和前面章节中的一致，
* 匹配 `argocd` 这个命名空间下面的具有 `"app.kubernetes.io/name": argocd-server-metrics` 这个 `label` 标签的 `Service`，`jobLabel` 表示用于检索 `job` 任务名称的标签，

## **查看 Service**

`ServiceMonitor` 创建完成了，可以查看一下关联的对应的 `Service` 对象，

```
kubectl get svc -n argocd --show-labels
NAME                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE   LABELS
argocd-metrics          ClusterIP   100.64.219.240   <none>        8082/TCP            18d   app.kubernetes.io/component=metrics,app.kubernetes.io/name=argocd-metrics,app.kubernetes.io/part-of=argocd
...
argocd-repo-server      ClusterIP   100.68.97.78     <none>        8081/TCP,8084/TCP   18d   app.kubernetes.io/component=repo-server,app.kubernetes.io/name=argocd-repo-server,app.kubernetes.io/part-of=argocd
...
argocd-server-metrics   ClusterIP   100.68.125.117   <none>        8083/TCP            18d   app.kubernetes.io/component=server,app.kubernetes.io/name=argocd-server-metrics,app.kubernetes.io/part-of=argocd
```


## **Add to dashboard** 

```
grafanaDashboards+:: {
	...
	 'argocd-dashboard.json': (import 'jam-dashboards/argocd-dashboard.json'),
}
```

## **配置 `PrometheusRule` (Applied after received metrics data)**


**`kustomize/templates/kube-prometheus/agocd_rules.json`**

```
{
  "groups": [
    {
      "name": "argocd.rules",
      "rules": [
        {
          "alert": "ArgoCDSyncProgress",
          "expr": "sum(argocd_app_health_status{health_status=\"Progressing\",namespace=\"argocd\"}) + sum(argocd_app_sync_status{sync_status=\"OutOfSync\",project=~\"(dev|integration|stage|jam).*\"}) > 0",
          "for": "30s",
          "labels": {
            "severity": "page",
            "source": "argocd"
          },
          "annotations": {
            "description": "ArgoCD start syncing applocations",
            "summary": "Deploying"
          }
        }
      ]
    }
  ]
}
```

### `kustomize/templates/kube-prometheus/template.jsonnet`

```
prometheusAlerts+:: {
       groups+: (import 'alerts/ct_rules.json').groups + (import 'alerts/argocd_rules.json').groups + (import 'alerts/jam_es_rules.json').groups,
    },

```

```
[~/sap-jam/jam-on-k8s]$ cd kustomize/templates/kube-prometheus && time docker run --rm -v $(pwd):$(pwd) --workdir $(pwd) quay.io/coreos/jsonnet-ci ./build.sh template.jsonnet
```

**`monitoring/kube-prometheus/prometheus-rules.yaml`**

```
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  labels:
    prometheus: k8s
    role: alert-rules
  name: prometheus-k8s-rules
  namespace: monitoring
spec:
  groups:
  ...
  - name: argocd.rules
    rules:
    - alert: ArgoCDSyncProgress
      annotations:
        description: ArgoCD start syncing applocations
        summary: Check configuration
      expr: sum(argocd_app_health_status{health_status="Progressing",namespace="argocd"}) > 0
      for: 1m
      labels:
        severity: page
        source: argocd
```

```
kubectl apply -f prometheus-rules.yaml
```

## **Add slack alerts fot argo deployment**


```
"global":
  "resolve_timeout": "5m"
  "smtp_hello": "sapjam-integration.com"
  "smtp_from": "...@sapjam-integration.com"    
  "smtp_smarthost": "email-smtp.eu-central-1.amazonaws.com:25" # 465 is implicit ssl port
  "smtp_auth_username": ""                  # stage stmp username and password
  "smtp_auth_password": ""
  "smtp_require_tls": true  
"receivers":
- "name": "jam-devops-email-alerts"
  "email_configs":
    - "send_resolved": true
      "headers":
        "Subject": 'Integration702-Alert{{ template "email.default.subject" . }}'
      "to": "receiver@sap.com"
- "name": "slack-deployment"
  "slack_configs":
  - "api_url": "https://hooks.slack.com/services/..."
    "channel": "multi-cloud-deployment"
    "send_resolved": true
    "text": "<!channel> \n`{{ .CommonAnnotations.instance }}` ArgoCD syncing {{ if eq .Status \"firing\" }}started :peperunner:{{ else }}done! :parrot_beer:{{ end }}"
    "title": "Deployment for {{ .CommonAnnotations.instance }}"
"route":
  "group_by":
  - "job"
  "group_interval": "5m"
  "group_wait": "30s"
  "receiver": "jam-devops-email-alerts"
  "repeat_interval": "24h"
  "routes":
  - "match":
      "alertname": "ArgoCDSyncProgress"
    "receiver": "slack-deployment"
  - "match":
      "severity": "critical"
    "receiver": "jam-devops-email-alerts"

"inhibit_rules":
- "source_match":
    "severity": "critical"
  "target_match_re":
    "severity": "warning|none"
  "equal":  ['prometheus']
- "source_match":
    "alertname": "KubeControllerManagerDown"
  "target_match_re":
    "alertname": "KubeSchedulerDown"
  "equal":  ["prometheus"]
- "source_match":
    "alertname": "KubeSchedulerDown"
  "target_match_re":
    "alertname": "KubeControllerManagerDown"
  "equal":  ['prometheus']
``` 
