# **第九节 通过kube-state-metrics添加CRD状态指标**

在 Kubernetes 中，自定义资源（CRD）是一种扩展 API 的机制，可以使用 Kubernetes APIServer 来存储和管理自定义对象。

本文我们将描述如何在不编写自定义资源注册表和自己构建 KSM 的情况下，根据自定义资源的状态来添加指标。

## **配置**

需要下面描述的 YAML 配置文件来定义你的自定义资源和要转换为指标的字段。

有两个参数需要用到：

* `--custom-resource-state-config` 在内联的 yaml 中
* `--custom-resource-state-config-file` 指定的配置文件路径

如果提供了两个标志，则内联配置将优先。

**当同一资源存在多个条目时，`kube-state-metrics` 将退出并报错，这包括引用不同 API 版本的配置**。

**除了指定 `--custom-resource-state-config*` 标志之外，还应该将复数形式的自定义资源种类添加到 `--resources` 标志中的暴露资源列表中**。

如果不指定 `--resources`，那么 `kube-state-metrics` 将考虑在 `--custom-resource-state-config*` 中配置的所有已知自定义资源和所有可用的默认 kubernetes 对象。

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kube-state-metrics
  namespace: kube-system
spec:
  template:
    spec:
      containers:
      - name: kube-state-metrics
        args:
          - --custom-resource-state-config
          -  |  # 在YAML文件中，| 允许将多行字符串作为标志值传递  # https://yaml-multiline.info
              spec:
                resources:
                  - groupVersionKind:
                      group: myteam.io
                      version: "v1"
                      kind: Foo
                    metrics:
                      - name: active_count
                        help: "Count of active Foo"
                        each:
                          type: Gauge
                          ...
          - --resources=certificatesigningrequests,configmaps,cronjobs,daemonsets,deployments,endpoints,foos,horizontalpodautoscalers,ingresses,jobs,limitranges,mutatingwebhookconfigurations,namespaces,networkpolicies,nodes,persistentvolumeclaims,persistentvolumes,poddisruptionbudgets,pods,replicasets,replicationcontrollers,resourcequotas,secrets,services,statefulsets,storageclasses,validatingwebhookconfigurations,volumeattachments,verticalpodautoscalers
```

也可以将 kube-state-metrics 配置为仅在自定义资源模式下运行，除了指定 `--custom-resource-state-config*` 标志之外，还可以将 `--custom-resource-state-only` 设置为 true。

使用此配置，`kube-state-metrics` 只会考虑在 `--custom-resource-state-config*` 中配置的已知自定义资源。

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kube-state-metrics
  namespace: kube-system
spec:
  template:
    spec:
      containers:
      - name: kube-state-metrics
        args:
          - --custom-resource-state-config
          -  |
              spec:
                resources:
                  - groupVersionKind:
                      group: myteam.io
                      version: "v1"
                      kind: Foo
                    metrics:
                      - name: active_count
                        help: "Count of active Foo"
                        each:
                          type: Gauge
                          ...
          - --custom-resource-state-only=true
```

注意：`customresource_group`、`customresource_version` 和 `customresource_kind` 这几个公共标签是保留的，将被 `groupVersionKind `字段中的值覆盖。

## 示例

下面我们将将使用以下自定义资源来进行说明：

```
kind: Foo
apiVersion: myteam.io/vl
metadata:
    annotations:
        bar: baz
        qux: quxx
    labels:
        foo: bar
    name: foo
spec:
    version: v1.2.3
    order:
        - id: 1
          value: true
        - id: 3
          value: false
    replicas: 1
status:
    phase: Pending
    active:
        type-a: 1
        type-b: 3
    conditions:
        - name: a
          value: 45
        - name: b
          value: 66
    sub:
        type-a:
            active: 1
            ready: 2
        type-b:
            active: 3
            ready: 4
    uptime: 43.21
```

### **单值**

配置如下：

```
kind: CustomResourceStateMetrics
spec:
  resources:
    - groupVersionKind:
        group: myteam.io
        kind: "Foo"
        version: "v1"
      metrics:
        - name: "uptime"
          help: "Foo uptime"
          each:
            type: Gauge
            gauge:
              path: [status, uptime]
```

使用上面的配置会产生如下所示指标：

```
kube_customresource_uptime{customresource_group="myteam.io", customresource_kind="Foo", customresource_version="v1"} 43.21
```

### 多指标

```
kind: CustomResourceStateMetrics
spec:
  resources:
    - groupVersionKind:
        group: myteam.io
        kind: "Foo"
        version: "v1"
      commonLabels:   # labels can be added to all metrics from a resource
        crd_type: "foo"
      labelsFromPath:  # 来自 path 路径的标签
        name: [metadata, name]  # 相当于：name=metadata.name
      metrics:
        - name: "ready_count"
          help: "Number Foo Bars ready"
          each:
            type: Gauge
            gauge:
              # 以对象或数组为目标将生成每个元素标签 FromPath 的指标，并且值与此路径相关
              path: [status, sub]

              # 如果 path 目标是一个对象，则对象的 key 将会被用作标签值
              # StateSet 类型不支持这一点，因为所有值都是true，这是多余的。
              labelFromKey: type
              # 可以解析特定于此路径的标签值
              labelsFromPath:
                active: [active]
              # 要用作指标值的实际字段，应为数字、布尔值或 RFC3339 时间戳字符串。
              valueFrom: [ready]
          commonLabels:
            custom_metric: "yes"
          labelsFromPath:
            # 整个对象可以通过前缀“*”复制到标签中
            # *任何内容都将被复制到标签中，首先是排序最高的 * 字符串
            "*": [metadata, labels]
            "**": [metadata, annotations]
            
            # 或者可以复制特定字段。这些字段将始终覆盖*s中的值
            name: [metadata, name]
            foo: [metadata, labels, foo]
```

然后会产生如下所示的指标：

```
kube_customresource_ready_count{customresource_group="myteam.io", customresource_kind="Foo", customresource_version="v1", active="1",custom_metric="yes",foo="bar",name="foo",bar="baz",qux="quxx",type="type-a"} 2
kube_customresource_ready_count{customresource_group="myteam.io", customresource_kind="Foo", customresource_version="v1", active="3",custom_metric="yes",foo="bar",name="foo",bar="baz",qux="quxx",type="type-b"} 4
```

## 指标类型

该配置支持 OpenMetrics 规范 中的三种指标。指标类型由 type 字段及其在类型特定结构中的特定配置指定。

### Gauge

Gauge 是表示的是当前的测量值，如当前使用的内存字节数或队列中的项目数，对于 gauge 来说，绝对值是大家感兴趣的东西。例如：

```
kind: CustomResourceStateMetrics
spec:
  resources:
    - groupVersionKind:
        group: myteam.io
        kind: "Foo"
        version: "v1"
      metrics:
        - name: "uptime"
          help: "Foo uptime"
          each:
            type: Gauge
            gauge:
              path: [status, uptime]  # status.uptime
```

使用上面的配置可以产生如下所示的指标：

```
kube_customresource_uptime{customresource_group="myteam.io", customresource_kind="Foo", customresource_version="v1"} 43.21
```

不过需要注意下类型转换。Gauge 生成 float64 类型的值，但自定义资源可以是各种类型，kube-state-metrics 对很多类型执行隐式类型转换，支持的类型包括：

* (u)int32/64、int、float32 和 byte 转换为 float64
* 如果 NilIsZero 为真，nil 一般会被映射为 0.0，否则会产生一个错误
* bool 类型 true 映射到 1.0，false 映射到 0.0
* 对于字符串，以下逻辑适用：
	* "true" 和 "yes" 被映射为 1.0，"false" 和 "no"被映射为 0.0（全部不区分大小写）
	* RFC3339 时间被解析为浮点时间戳
	* 最后使用 https://pkg.go.dev/strconv#ParseFloat 将字符串解析为浮点数，它应该支持所有常见的数字格式，如果失败，则会产生错误

Kubernetes 控制器上的状态条件示例：

```
kind: CustomResourceStateMetrics
spec:
  resources:
  - groupVersionKind:
      group: myteam.io
      kind: "Foo"
      version: "v1"
    labelsFromPath:
      name:
      - metadata
      - name
      namespace:
      - metadata
      - namespace
    metrics:
    - name: "foo_status"
      help: "status condition "
      each:
        type: Gauge
        gauge:
          path: [status, conditions]
          labelsFromPath:
            type: ["type"]
          valueFrom: ["status"]
```

这适用于根据 kubernetes api (https://pkg.go.dev/k8s.io/apimachinery/pkg/apis/meta/v1#Condition) 公开状态条件的 kubernetes 控制器 CR：

```
status:
  conditions:
    - lastTransitionTime: "2019-10-22T16:29:31Z"
      status: "True"
      type: Ready
```

可以产生如下所示的指标：

```
kube_customresource_foo_status{customresource_group="myteam.io", customresource_kind="Foo", customresource_version="v1", type="Ready"} 1.0
```

### StateSet

StateSet 表示一系列相关的布尔值，也称为 bitset。如果需要对 ENUM 进行编码，则可以通过 StateSet 来完成。例如：

```
kind: CustomResourceStateMetrics
spec:
  resources:
    - groupVersionKind:
        group: myteam.io
        kind: "Foo"
        version: "v1"
      metrics:
        - name: "status_phase"
          help: "Foo status_phase"
          each:
            type: StateSet
            stateSet:
              labelName: phase
              path: [status, phase]
              list: [Pending, Bar, Baz]
```

StateSet 类型的指标将为每个资源的列表中定义的每个值生成一个指标，如果该值与列表中的值匹配，则该值将为 1。会生成如下所示的指标：

```
kube_customresource_status_phase{customresource_group="myteam.io", customresource_kind="Foo", customresource_version="v1", phase="Pending"} 1
kube_customresource_status_phase{customresource_group="myteam.io", customresource_kind="Foo", customresource_version="v1", phase="Bar"} 0
kube_customresource_status_phase{customresource_group="myteam.io", customresource_kind="Foo", customresource_version="v1", phase="Baz"} 0
```

### Info

Info 指标用于暴露文本信息，这些信息在进程生命周期内不应更改，常见示例是应用程序的版本、修订控制提交和编译器的版本。 Info 类型的指标的值始终为 1，例如：

```
kind: CustomResourceStateMetrics
spec:
  resources:
    - groupVersionKind:
        group: myteam.io
        kind: "Foo"
        version: "v1"
      metrics:
        - name: "version"
          help: "Foo version"
          each:
            type: Info
            info:
              labelsFromPath:
                version: [spec, version]
```
会产生如下所示的指标：

```
kube_customresource_version{customresource_group="myteam.io", customresource_kind="Foo", customresource_version="v1", version="v1.2.3"} 1
```

### 命名

默认指标名称带有前缀以避免与其他指标冲突。默认情况下，使用 `kube_` 的指标前缀与自定义资源的 `group+version+kind `连接，你可以使用 `metricNamePrefix` 字段覆盖此行为。

```
kind: CustomResourceStateMetrics
spec:
  resources:
    - groupVersionKind: ...
      metricNamePrefix: myteam_foos
      metrics:
        - name: uptime
          ...
```

会产生如下所示的指标：

```
myteam_foos_uptime{customresource_group="myteam.io", customresource_kind="Foo", customresource_version="v1"} 43.21
```

要完全省略命名空间空间和/或子系统，请将它们设置为空字符串：

```
kind: CustomResourceStateMetrics
spec:
  resources:
    - groupVersionKind: ...
      metricNamePrefix: ""
      metrics:
        - name: uptime
          ...
```

会产生如下的指标：

```
uptime{customresource_group="myteam.io", customresource_kind="Foo", customresource_version="v1"} 43.21
```

### 日志

如果已注册指标路径但未在自定义资源上找到，则会记录错误日志。对于某些资源，这可能会产生大量无用的日志数据，可以使用资源或指标上的 errorLogV 设置指标或资源的错误日志详细程度：

```
kind: CustomResourceStateMetrics
spec:
  resources:
    - groupVersionKind: ...
      errorLogV: 0  # 0 = default for errors
      metrics:
        - name: uptime
          errorLogV: 10  # only log at high verbosity
```

### **路径语法**

Path 路径被指定为字符串列表，每个字符串都是一个 path 片段，根据自定义资源的数据动态解析。如果缺少 path 的任何部分，则结果为零。例如：

```
# 简单路径查找
[spec, replicas]                         # spec.replicas == 1

# 数组索引
[spec, order, "0", value]                # spec.order[0].value = true

# 通过 key = value 在列表中查找元素
[status, conditions, "[name=a]", value]  # status.conditions[0].value = 45

# 如果要匹配的值是数字或布尔值，则将该值作为数字或布尔值进行比较
[status, conditions, "[value=66]", name]  # status.conditions[1].name = "b"
```


