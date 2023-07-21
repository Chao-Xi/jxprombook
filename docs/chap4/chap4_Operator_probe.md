# **Prometheus Operator中探针的使用（Blackbox Exporter）**

需要对服务器和线上业务进行一些探针来定时拨测，用于对服务的存活性进行监控与告警。很早以前就知道prometheus社区提供了 blackbox 的探针方案，但一直没有关注，正好趁这次机会了解一下。

Blackbox Exporter 是 Prometheus 社区提供的官方黑盒监控解决方案，其允许用户通过：HTTP, HTTPS, DNS, TCP, ICMP 和 gRPC.的方式对网络进行探测。

目前 proemtheus operator 中的 probe 资源已实现对 blackbox-exporter 的支持，本文的介绍的所有探针也均在 probe中实现。

## 环境准备

在使用之前，须确保你的 k8s集群内已经部署了 prometheus operator和prometheus-blackbox-exporter，如果没有安装，可使用 helm命令直接部署。

```
# 添加 promethues 社区 helm 源 并更新
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update prometheus-community

# 安装 prometheus
helm install prometheus-community/prometheus-operator

# 安装 blackbox-exporter
helm install prometheus-community/prometheus-blackbox-exporter
```

安装成功后，需手动配置 black-exporter的 configmap，引入探针模块。

> 提示：可以在`prometheus-blackbox-exporter`的 `helm values` 中配置。文章为了直观，所以直接修改 configmap

```
 blackbox.yaml: |
    modules:
      dns:
        dns:
          preferred_ip_protocol: ip4
          query_name: kubernetes.default.svc.cluster.local
          transport_protocol: tcp
        prober: dns
      grpc:
        grpc:
          preferred_ip_protocol: ip4
          tls: true
        prober: grpc
      grpc_plain:
        grpc:
          service: service1
          tls: false
        prober: grpc
      http_2xx:
        http:
          follow_redirects: true
          preferred_ip_protocol: ip4
          valid_http_versions:
          - HTTP/1.1
          - HTTP/2.0
        prober: http
        timeout: 5s
      http_post_2xx:
        http:
          method: POST
          preferred_ip_protocol: ip4
          valid_http_versions:
          - HTTP/1.1
          - HTTP/2
        prober: http
        headers:
          content-type: application/json
        body: '{"k": "v"}'   //自定义的 body 数据
        timeout: 5s
      ping:
        icmp:
          preferred_ip_protocol: ip4
        prober: icmp
        timeout: 5s
      tcp_connect:
        prober: tcp
        timeout: 5s
```

以上配置，让 blackbox-exporter 加载了完整的网络探针模块。

## 拨测

probe实现了对 blackbox-exporter 的配置管理，极大的简

### 1. 拨测 ICMP

```
kind: Probe
apiVersion: monitoring.coreos.com/v1
metadata:
  name: icmp-probe
  namespace: kubegems-monitoring
spec:
  interval: 60s
  module: ping
  prober:
    url: prometheus-blackbox-exporter.kubegems-monitoring.svc.cluster.local:9115
  targets:
    staticConfig:
      static:
      - 114.114.114.114
      - baidu.com
```

通过查询PromQL **`count by (__name__) ({job="probe/kubegems-monitoring/icmp-probe"})`**，我们可以得到如下指标

```
probe_dns_lookup_time_seconds{}  // icmp中域名解析耗时
probe_duration_seconds{}  // 拨测耗时
probe_icmp_duration_seconds{}  // icmp 的耗时      
probe_success{}  //拨测结果
```

### 拨测 DNS

```
kind: Probe
apiVersion: monitoring.coreos.com/v1
metadata:
  name: dns-probe
  namespace: kubegems-monitoring
spec:
  interval: 60s
  module: dns
  prober:
    url: prometheus-blackbox-exporter.kubegems-monitoring.svc.cluster.local:9115
  targets:
    staticConfig:
      static:
      - kubegems.io
```

通过查询PromQL `count by (__name__) ({job="probe/kubegems-monitoring/dns-probe"})`，我们可以得到如下指标

```
probe_dns_additional_rrs{}           // 附加记录列表中的条目数量
probe_dns_answer_rrs{}                // 响应记录列表中的条目数量
probe_dns_authority_rrs{}             // 权威记录列表中的条目数量
probe_dns_duration_seconds{}     // dns解析耗时
```

### 拨测 TCP

```
kind: Probe
apiVersion: monitoring.coreos.com/v1
metadata:
  name: tcp-probe
  namespace: kubegems-monitoring
spec:
  interval: 60s
  module: tcp_connect
  prober:
    url: prometheus-blackbox-exporter.kubegems-monitoring.svc.cluster.local:9115
  targets:
    staticConfig:
      static:
      - kubegems.io:443
```

通过查询PromQL `count by (__name__) ({job="probe/kubegems-monitoring/tcp-probe"})`，我们可以得到如下指标

```
probe_dns_lookup_time_seconds{}
probe_duration_seconds{}
probe_failed_due_to_regex{}
probe_ip_addr_hash{}
probe_ip_protocol{}
probe_success{}
```

### 拨测 HTTP GET

```
kind: Probe
apiVersion: monitoring.coreos.com/v1
metadata:
  name: http-probe
  namespace: kubegems-monitoring
spec:
  interval: 60s
  module: http_2xx
  prober:
    path: /probe
    url: prometheus-blackbox-exporter.kubegems-monitoring.svc.cluster.local:9115
  targets:
    staticConfig:
      static:
      - https://kubegems.io
```

通过查询PromQL `count by (__name__) ({job="probe/kubegems-monitoring/http-probe"})`，我们可以得到如下指标

```
probe_dns_lookup_time_seconds{}  // http拨测中的 dns 查询耗时
probe_duration_seconds{}
probe_failed_due_to_regex{}
probe_http_content_length{}
probe_http_duration_seconds{}    // http handler 各阶段处理耗时
probe_http_last_modified_timestamp_seconds{}
probe_http_redirects{}
probe_http_ssl{}    // 是否启用 https
probe_http_status_code{}   // http 状态码
probe_http_uncompressed_body_length{}
probe_http_version{}
probe_ssl_earliest_cert_expiry{}  // ssl证书过期时间
probe_ssl_last_chain_expiry_timestamp_seconds{}
probe_ssl_last_chain_info{}
probe_success{}
probe_tls_version_info{}   // tls版本
```

### 拨测 GRPC

```
kind: Probe
apiVersion: monitoring.coreos.com/v1
metadata:
  name: grpc-probe
  namespace: kubegems-monitoring
spec:
  interval: 60s
  module: grpc_plain
  prober:
    url: prometheus-blackbox-exporter.kubegems-monitoring.svc.cluster.local:9115
  targets:
    staticConfig:
      static:
      - 172.16.23.86:30590
```


通过查询PromQL `count by (__name__) ({job="probe/kubegems-monitoring/grpc-probe"})`，

我们可以得到如下指标

```

probe_dns_lookup_time_seconds{}
probe_duration_seconds{}
probe_grpc_duration_seconds{}
probe_grpc_healthcheck_response{}
probe_grpc_ssl{}
probe_grpc_status_code{}
probe_ssl_earliest_cert_expiry{}
probe_success{}
```

