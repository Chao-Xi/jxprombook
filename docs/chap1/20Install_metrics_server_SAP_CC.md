# **第五节 Install Kubernetes Metrics Server on SAP Converged Cloud**

## **1 Project Introduction**

### **1 Bcakground 1: Sap Converged Cloud**

**SAP Converged Cloud**: From Playground to Production on one Infrastructure-as-a-Service (IaaS), powered by Kubernetes, OpenStack, SAP HANA and Monsoon Automation

### **2 Bcakground 2: Why Metrics servers not heaspter**

**RETIRED**: Heapster is now retired. . We will not be making changes to Heapster.

The following are potential migration paths for Heapster functionality:

* **For basic CPU/memory HPA metrics:** Use metrics-server.
* **For general monitoring**: Consider a third-party monitoring pipeline that can gather Prometheus-formatted metrics. The kubelet exposes all the metrics exported by Heapster in Prometheus format.



### **3 Prvious: Install Tiller Server for the cluster**

**Check which cluster to install tiller server**

```
$ kubectl config current-context
jam  
```

#### **initialize Helm on both client and server**

**Helm 2**

```
$ helm init
$HELM_HOME has been configured at /Users/.../.helm.

Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.

Please note: by default, Tiller is deployed with an insecure 'allow unauthenticated users' policy.
To prevent this, run `helm init` with the --tiller-tls-verify flag.
```

```
$ kubectl get pods --all-namespaces
NAMESPACE     NAME                             READY   STATUS             RESTARTS   AGE
kube-system   tiller-deploy-595f59d579-nszzh   1/1     Running            0          35s
```

## **2 Install Metrics Server by `helm`**

### **2-1" fetch"metrics server package**

```
helm fetch:     download a chart to your local directory to view
```

```
$ helm fetch stable/metrics-server
$ tar -xvf metrics-server-2.8.2.tgz
x metrics-server/Chart.yaml
x metrics-server/values.yaml
x metrics-server/templates/NOTES.txt
x metrics-server/templates/_helpers.tpl
x metrics-server/templates/aggregated-metrics-reader-cluster-role.yaml
x metrics-server/templates/auth-delegator-crb.yaml
x metrics-server/templates/cluster-role.yaml
x metrics-server/templates/metric-server-service.yaml
x metrics-server/templates/metrics-api-service.yaml
x metrics-server/templates/metrics-server-crb.yaml
x metrics-server/templates/metrics-server-deployment.yaml
x metrics-server/templates/metrics-server-serviceaccount.yaml
x metrics-server/templates/pdb.yaml
x metrics-server/templates/psp.yaml
x metrics-server/templates/role-binding.yaml
x metrics-server/templates/tests/test-version.yaml
x metrics-server/.helmignore
x metrics-server/README.md
x metrics-server/ci/ci-values.yaml

$ helm install ./metrics-server --namespace kube-system --name metric
```

* `--namespace`: `special namespaces`
* `--name`: `special release name`

```
$ helm install ./metrics-server --namespace kube-system

Error: no available release name found
```

```
$ helm install ./metrics-server --namespace kube-system --name metric

Error: release metric failed: namespaces "kube-system" is forbidden: User "system:serviceaccount:kube-system:default" cannot get namespaces in the namespace "kube-system"
```


### **2-2 Looks like we have `Service-account` forbidden Problem**


#### **So create `tiller` SA accound and role**

```
$ kubectl create serviceaccount --namespace kube-system tiller
serviceaccount/tiller created

$ kubectl get sa --all-namespaces
NAMESPACE     NAME                                 SECRETS   AGE
kube-system   tiller                               1         28s

$ kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller

clusterrolebinding.rbac.authorization.k8s.io/tiller-cluster-rule created

$ kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller

clusterrolebinding.rbac.authorization.k8s.io/tiller-cluster-rule created

$ kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'
deployment.extensions/tiller-deploy patched
```  

### **2-3 Install The metrics server**

```
$ helm install ./metrics-server --namespace kube-system --name metric
NAME:   metric
LAST DEPLOYED: Fri Jul 19 17:25:29 2019
NAMESPACE: kube-system
STATUS: DEPLOYED

RESOURCES:
==> v1/ClusterRole
NAME                                     AGE
system:metric-metrics-server             2s
system:metrics-server-aggregated-reader  2s

==> v1/ClusterRoleBinding
NAME                                         AGE
metric-metrics-server:system:auth-delegator  2s
system:metric-metrics-server                 2s

==> v1/Deployment
NAME                   READY  UP-TO-DATE  AVAILABLE  AGE
metric-metrics-server  0/1    1           0          2s

==> v1/Pod(related)
NAME                                    READY  STATUS             RESTARTS  AGE
metric-metrics-server-77df84fbdd-5bb9c  0/1    ContainerCreating  0         2s

==> v1/Service
NAME                   TYPE       CLUSTER-IP     EXTERNAL-IP  PORT(S)  AGE
metric-metrics-server  ClusterIP  198.18.216.71  <none>       443/TCP  2s

==> v1/ServiceAccount
NAME                   SECRETS  AGE
metric-metrics-server  1        2s

==> v1beta1/APIService
NAME                    AGE
v1beta1.metrics.k8s.io  2s

==> v1beta1/RoleBinding
NAME                               AGE
metric-metrics-server-auth-reader  2s


NOTES:
The metric server has been deployed.

In a few minutes you should be able to list metrics using the following
command:

  kubectl get --raw "/apis/metrics.k8s.io/v1beta1/nodes"
```


```
$ kubectl get pods -n kube-system`
NAME                                     READY   STATUS    RESTARTS   AGE
tiller-deploy-c5c655fc5-zldr4            1/1     Running   0          1m
metric-metrics-server-77df84fbdd-5bb9c   0/1     Running   0          30s
```


## **3 Fix problem**

Looks we have `unable to fetch metrics from Kubelet` problem here

```
$ kubectl logs -f metric-metrics-server-77df84fbdd-5bb9c -n kube-system
I0719 09:25:33.716791       1 serving.go:273] Generated self-signed cert (/tmp/apiserver.crt, /tmp/apiserver.key)
[restful] 2019/07/19 09:25:34 log.go:33: [restful/swagger] listing is available at https://:8443/swaggerapi
[restful] 2019/07/19 09:25:34 log.go:33: [restful/swagger] https://:8443/swaggerui/ is mapped to folder /swagger-ui/
I0719 09:25:34.884929       1 serve.go:96] Serving securely on [::]:8443
E0719 09:26:34.898366       1 manager.go:111] unable to fully collect metrics: [unable to fully scrape metrics from source kubelet_summary:jam-m2xlarge-fphgs: unable to fetch metrics from Kubelet jam-m2xlarge-fphgs (jam-m2xlarge-fphgs.novalocal): Get https://jam-m2xlarge-fphgs.novalocal:10250/stats/summary/: dial tcp: lookup jam-m2xlarge-fphgs.novalocal on 198.18.128.2:53: no such host, unable to fully scrape metrics from source kubelet_summary:jam-m2xlarge-6zqkq: unable to fetch metrics from Kubelet jam-m2xlarge-6zqkq (jam-m2xlarge-6zqkq.novalocal): Get https://jam-m2xlarge-6zqkq.novalocal:10250/stats/summary/: dial tcp: lookup jam-m2xlarge-6zqkq.novalocal on 198.18.128.2:53: no such host, unable to fully scrape metrics from source kubelet_summary:jam-m2xlarge-sxrj8: unable to fetch metrics from Kubelet jam-m2xlarge-sxrj8 (jam-m2xlarge-sxrj8.novalocal): Get https://jam-m2xlarge-sxrj8.novalocal:10250/stats/summary/: dial tcp: lookup jam-m2xlarge-sxrj8.novalocal on 198.18.128.2:53: no such host]


[restful] 2019/07/19 09:28:30 log.go:33: [restful/swagger] listing is available at https://:8443/swaggerapi
[restful] 2019/07/19 09:28:30 log.go:33: [restful/swagger] https://:8443/swaggerui/ is mapped to folder /swagger-ui/
I0719 09:28:30.854420       1 serve.go:96] Serving securely on [::]:8443
E0719 09:28:56.628369       1 reststorage.go:129] unable to fetch node metrics for node "jam-m2xlarge-6zqkq": no metrics known for node
E0719 09:28:56.628399       1 reststorage.go:129] unable to fetch node metrics for node "jam-m2xlarge-fphgs": no metrics known for node
E0719 09:28:56.628404       1 reststorage.go:129] unable to fetch node metrics for node "jam-m2xlarge-sxrj8": no metrics known for node
```

我们可以发现 `Pod` 中出现了一些错误信息：`xxx: no such host`，**我们看到这个错误信息一般就可以确定是 `DNS` 解析不了造成的，我们可以看到 `metrics-server` 会通过 `kubelet` 的 `10250` 端口获取信息，使用的是 `hostname`**，

我们部署集群的时候在节点的 `/etc/hosts `里面添加了节点的 `hostname` 和 `ip ` 的映射，但是是我们的 `metrics-server` 的 `Pod` 内部并没有这个 `hosts` 信息，当然也就不识别 `hostname `

**要解决这个问题，有两种方法：**

* 第一种方法就是在集群内部的 DNS 服务里面添加上 `hostname` 的解析
* 另外一种方法就是在 `metrics-server` 的启动参数中修改`kubelet-preferred-address-types`参数
 * 我们可以添加一个`--kubelet-insecure-tls`参数跳过证书校验 

### **3-1 Edit metric-server deployment to add the flags**


```
$ kubectl edit deployment metric-metrics-server -n kube-system
deployment.extensions/metric-metrics-server edited
```

**8 spaces**

```
args:
- --kubelet-insecure-tls
- --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
```

![Alt Image Text](../images/20_1.png "Body image")

**Please use 4 spaces to replace one tap, otherwise you may encounter unident problem**

```
$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                     READY   STATUS             RESTARTS   AGE
kube-system   metric-metrics-server-77df84fbdd-5bb9c   0/1     Terminating        9          26m
kube-system   metric-metrics-server-78996fbfbf-plcww   1/1     Running            0          1m
```

```
$ kubectl top nodes
NAME                 CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
jam-m2xlarge-6zqkq   39m          0%     948Mi           5%
jam-m2xlarge-fphgs   62m          0%     825Mi           5%
jam-m2xlarge-sxrj8   47m          0%     613Mi           3%
```

**Now it works fine**







