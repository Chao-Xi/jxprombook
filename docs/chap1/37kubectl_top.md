# **从kubectl top看K8S监控原理**

## **1 前言**

kubectl top 可以很方便地查看node、pod的实时资源使用情况：如CPU、内存。这篇文章会介绍其数据链路和实现原理，同时借`kubectl top` 阐述 k8s 中的监控体系，窥一斑而知全豹。最后会解释常见的一些问题：

* kubectl top 为什么会报错？
* kubectl top node 怎么计算，和节点上直接 top 有什么区别？
* kubectl top pod 怎么计算，包含 pause 吗？
* kubectl top pod 和exec 进入 pod 后看到的 top 不一样？
* kubectl top pod 和 docker stats得到的值为什么不同？

## **2 使用**

kubectl top 是基础命令，但是需要部署配套的组件才能获取到监控值

* 1.8以下：部署 `heapter`
* 1.8以上：部署 `metric-server`

### **2-1 安装**

```
helm install metric ./metrics-server --namespace kube-system

kubectl edit deployment metric-metrics-server -n kube-system
...
args:
- --kubelet-insecure-tls
- --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
...
```

![Alt Image Text](../images/37_1.png "Body image")

```
$ kubectl top nodes
NAME             CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
docker-desktop   499m         12%    2198Mi          27% 
```

**kubectl top pod: 查看 pod 的使用情况**

```
$ kubectl top pods
NAME                    CPU(cores)   MEMORY(bytes)   
dark-57c5c58bf8-llxqj   1m           4Mi
```

```
$ kubectl top pod kfk-zookeeper-0 -n kafka 
NAME              CPU(cores)   MEMORY(bytes)   
kfk-zookeeper-0   3m           187Mi

$ kubectl top pod kfk-zookeeper-0 -n kafka --containers
POD               NAME        CPU(cores)   MEMORY(bytes)   
kfk-zookeeper-0   zookeeper   3m           187Mi 
```

指标含义：

* 和`k8s`中的`request、limit`一致，`CPU`单位`100m=0.1 `内存单位`1Mi=1024Ki`
* `pod`的内存值是其实际使用量，也是做`limit`限制时判断`oom`的依据。`pod`的使用量等于其所有业务容器的总和，不包括 `pause` 容器，值等于`cadvisr`中的`container_memory_working_set_bytes`指标
* **`node`的值并不等于该`node` 上所有` pod` 值的总和，也不等于直接在机器上运行` top` 或 `free` 看到的值**

## **3 实现原理**

### **3.1 数据链路**

`kubectl top` 、 `k8s dashboard` 以及 `HPA `等调度组件使用的数据是一样，数据链路如下：

![Alt Image Text](../images/37_2.png "Body image")

**使用 `heapster` 时：`apiserver` 会直接将`metric`请求通过` proxy `的方式转发给集群内的 `hepaster` 服务。**

![Alt Image Text](../images/37_3.png "Body image")

**而使用 `metrics-server` 时：`apiserver`是通过`/apis/metrics.k8s.io/`的地址访问`metric`**

![Alt Image Text](../images/37_4.png "Body image")

**这里可以对比下`kubect get pod`时的日志：**

![Alt Image Text](../images/37_5.png "Body image")

### **3.2 metric api**

可以发现，`heapster`使用的是 `proxy `转发，**而 `metric-server `和普通 `pod`都是使用 `api/xx` 的资源接口**，`heapster`采用的这种 `proxy `方式是有问题的：

* `proxy`只是代理请求，一般用于问题排查，不够稳定，且版本不可控
* `heapster`的接口不能像`apiserver`一样有完整的鉴权以及`client`集成，两边都维护的话代价高，如`generic apiserver`）
* `pod` 的监控数据是核心指标（HPA调度），应该和 `pod` 本身拥有同等地位，即 `metric`应该作为一种资源存在，如`metrics.k8s.io` 的形式，称之为 `Metric Api`

于是官方从 1.8 版本开始逐步废弃 `heapster`，并提出了上边`Metric api `的概念，而`metrics-server` 就是这种概念下官方的一种实现，用于从 `kubelet`获取指标，替换掉之前的` heapster`

### **3.3 kube-aggregator**

有了metrics-server组件，采集到了需要的数据，也暴露了接口，但走到这一步和 heapster 其实没有区别，最关键的一步就是如何将打到`apiserver的/apis/metrics.k8s.io`请求转发给`metrics-server`组件？

解决方案就是：[kube-aggregator](https://github.com/kubernetes/kube-aggregator)。

`kube-aggregator`是对 `apiserver `的有力扩展，它允许k8s的开发人员编写一个自己的服务，并把这个服务注册到k8s的`ap`i里面，即扩展 `API`，`metric-server`其实在` 1.7`版本就已经完成了，只是在等`kube-aggregator`的出现。

kube-aggregator是 apiserver 中的实现，有些 k8s 版本默认没开启，你可以加上这些[配置](https://k8smeetup.github.io/docs/tasks/access-kubernetes-api/configure-aggregation-layer/#%E5%90%AF%E7%94%A8-apiserver-%E7%9A%84%E6%A0%87%E8%AE%B0)
来开启。他的核心功能是动态注册、发现汇总、安全代理。

![Alt Image Text](../images/37_6.png "Body image")

如 `metric-server` 注册 `pod` 和 `node` 时:

![Alt Image Text](../images/37_7.png "Body image")

### **3.4 监控体系**

在提出 metric api 的概念时，官方也提出了新的监控体系，监控资源被分为了2种：

* `Core metrics`(核心指标)：从 `Kubelet`、`cAdvisor` 等获取度量数据，再由`metrics-server` 提供给` Dashboard`、`HPA `控制器等使用。
* `Custom Metrics`(自定义指标)：由 `Prometheus Adapter` 提供` API custom.metrics.k8s.io`，由此可支持任意`Prometheus`采集到的指标。


![Alt Image Text](../images/37_8.png "Body image")

核心指标只包含`node`和`pod`的`cpu`、内存等，一般来说，核心指标作HPA已经足够，但如果想根据自定义指标:如请求`qps/5xx`错误数来实现HPA，就需要使用自定义指标了。

目前`Kubernetes`中自定义指标一般由`Prometheus`来提供，再利用`k8s-prometheus-adpater`聚合到`apiserver`，实现和核心指标（`metric-server`)同样的效果。

### **3.5 kubelet**

前面提到，无论是 `heapster`还是 `metric-server`，都只是数据的中转和聚合，两者都是调用的 `kubelet` 的 `api `接口获取的数据，而 `kubelet` 代码中实际采集指标的是 `cadvisor` 模块，你可以在 `node` 节点访问 `10255` 端口 （`read-only-port`)获取监控数据：

* `Kubelet Summary metrics: 127.0.0.1:10255/metrics`，暴露` node、pod `汇总数据
* `Cadvisor metrics: 127.0.0.1:10255/metrics/cadvisor`，暴露 `container` 维度数据

示例，容器的内存使用量：

![Alt Image Text](../images/37_9.png "Body image")

kubelet虽然提供了` metric` 接口，但实际监控逻辑由内置的`cAdvisor`模块负责，演变过程如下：

* 从`k8s 1.6`开始，`kubernetes`将`cAdvisor`开始集成在`kubelet`中，不需要单独配置
* 从`k8s 1.7`开始，`Kubelet metrics API` 不再包含 `cadvisor metrics`，而是提供了一个独立的 `API` 接口来做汇总
* 从`k8s 1.12`开始，`cadvisor` 监听的端口在`k8s`中被删除，所有监控数据统一由`Kubelet`的`API`提供

到这里为止，`k8s`范围内的监控体系就结束了，如果你想继续了解cadvisor和 cgroup 的内容，可以向下阅读

### **3.6 cadvisor**

cadvisor由谷歌开源，使用Go开发，项目地址也是`google/cadvisor`，`cadvisor`不仅可以搜集一台机器上所有运行的容器信息，包括`CPU`使用情况、内存使用情况、网络吞吐量及文件系统使用情况，还提供基础查询界面和http接口，方便其他组件进行数据抓取。在K8S中集成在Kubelet里作为默认启动项，k8s官方标配。

cadvisor 拿到的数据结构示例：

![Alt Image Text](../images/37_10.png "Body image")

**核心逻辑：**

通过`new`出来的`memoryStorage`以及`sysfs`实例，创建一个`manager`实例，`manager`的`interface`中定义了许多用于获取容器和`machine`信息的函数

![Alt Image Text](../images/37_11.png "Body image")

cadvisor的指标解读：[cgroup-v1](https://www.kernel.org/doc/Documentation/cgroup-v1/memory.txt)

`cadvisor`获取指标时实际调用的是 `runc/libcontainer`库，而`libcontainer`是对` cgroup`文件 的封装，即 `cadvsio`r也只是个转发者，它的数据来自于`cgroup`文件。

### **3.7 cgroup**

cgroup文件中的值是监控数据的最终来源，如


* `mem usage`的值，来自于 `/sys/fs/cgroup/memory/docker/[containerId]/memory.usage_in_bytes`
* 如果没限制内存，`Limit = machine_mem`，否则来自于`/sys/fs/cgroup/memory/docker/[id]/memory.limit_in_bytes`
* 内存使用率 `= memory.usage_in_bytes/memory.limit_in_bytes`

一般情况下，cgroup文件夹下的内容包括CPU、内存、磁盘、网络等信息：

![Alt Image Text](../images/37_12.png "Body image")

如memory下的几个常用的指标含义：

![Alt Image Text](../images/37_13.png "Body image")

`memory.stat`中的信息是最全的：

![Alt Image Text](../images/37_14.png "Body image")

原理到这里结束，这里解释下最开始的kubectl top 的几个问题：

## **四. 问题**

### **4.1 kubectl top 为什么会报错**

一般情况下 top 报错有以下几种，可以 `kubectl top pod -v=10`看到具体的调用日志:

* 没有部署 `heapster` 或者 `metric-server`，或者`pod`运行异常，可以排查对应 `pod`日志
* 要看的`pod `刚刚建出来，还没来得及采集指标，报 `not found `错误，默认`1 `分钟
* 以上两种都不是，可以检查下`kubelet` 的 `10255 `端口是否开放，默认情况下会使用这个只读端口获取指标，也可以在`heapster`或`metric-server`的配置中增加证书，换成 `10250` 认证端口

### **4.2 kubectl top pod 内存怎么计算，包含 pause容器吗**

每次启动` pod`，都会有一个 `pause` 容器，既然是容器就一定有资源消耗（一般在 2-3M 的内存），`cgroup` 文件中，业务容器和` pause ` 容器都在同一个 pod的文件夹下。

**但 `cadvisor` 在查询 `pod` 的内存使用量时，是先获取了 `pod` 下的`container`列表，再逐个获取`container`的内存占用，**不过这里的 `container `列表并没有包含` pause`，**因此最终` top pod` 的结果也不包含 `pause` 容器**

### **pod 的内存使用量计算**

**kubectl top pod 得到的内存使用量，并不是`cadvisor` 中的`container_memory_usage_bytes`，而是`container_memory_working_set_bytes`，计算方式为：**

* `container_memory_usage_bytes == container_memory_rss + container_memory_cache + kernel memory`
* `container_memory_working_set_bytes = container_memory_usage_bytes – total_inactive_file`（未激活的匿名缓存页）

`container_memory_working_set_bytes`是容器真实使用的内存量，也是`limit`限制时的 `oom` 判断依据

`cadvisor `中的 `container_memory_usage_bytes`对应 `cgroup` 中的 `memory.usage_in_bytes`文件，但`container_memory_working_set_bytes`并没有具体的文件，他的计算逻辑在 `cadvisor` 的代码中，如下：

![Alt Image Text](../images/37_15.png "Body image")

同理，node 的内存使用量也是`container_memory_working_set_bytes`

### **4.3 kubectl top node 怎么计算，和节点上直接 top 有什么区别**

`kubectl top node`得到的 `cpu` 和内存值，并不是节点上所有 `pod` 的总和，不要直接相加。`top node`是机器上`cgroup`根目录下的汇总统计

![Alt Image Text](../images/37_16.png "Body image")

**在机器上直接 top命令看到的值和` kubectl top node `不能直接对比，因为计算逻辑不同，如内存，大致的对应关系是(前者是机器上 top，后者是kubectl top)**:

```
rss + cache = (in)active_anon + (in)active_file
```

### **4.4 kubectl top pod 和exec 进入 pod 后看到的 top 不一样**

top命令的差异和上边 一致，无法直接对比，同时，就算你对 `pod` 做了`limit` 限制，`pod` 内的 `top` 看到的内存和 `cpu`总量仍然是机器总量，并不是`pod` 可分配量

* 进程的`RSS`为进程使用的所有物理内存（`file_rss＋anon_rss`），即`Anonymous pages＋Mapped apges`（包含共享内存）
* `cgroup RSS`为（`anonymous and swap cache memory`），不包含共享内存。两者都不包含`file cache`

### **4.5 kubectl top pod 和 docker stats得到的值为什么不同？**

`docker stats dockerID` 可以看到容器当前的使用量：

![Alt Image Text](../images/37_17.png "Body image")

如果你的 `pod`中只有一个 `container`，你会发现`docker stats `值不等于`kubectl top` 的值，既不等于` container_memory_usage_bytes`，也不等于`container_memory_working_set_bytes`。

**因为`docker stats` 和 `cadvisor` 的计算方式不同，总体值会小于` kubectl top`**：计算逻辑是：

```
docker stats = container_memory_usage_bytes - container_memory_cache
```

## **五. 后记**

一般情况下，我们并不需要时刻关心node 或 pod 的使用量，因为有集群自动扩缩容(cluster-autoscaler)和pod 水平扩缩容（`HPA`）来应对这两种资源变化，资源指标的意义更适合使用`prometheus`来持久化` cadvisor `的数据，用于回溯历史或者发送报警。

其他补充：

* 虽然`kubectl top help`中显示支持`Storage`，但直到 1.16 版本仍然不支持
* 1.13 之前需要 heapster，1.13 以后需要`metric-server`，这部分`kubectl top help`的输出 有误，里面只提到了heapster
* `k8s dashboard` 中的监控图默认使用的是` heapster`，切换为 `metric-server`后数据会异常，需要多部署一个`metric-server-scraper` 的 `pod` 来做接口转换，具体参考 pr：[https://github.com/kubernetes/dashboard/pull/3504](https://github.com/kubernetes/dashboard/pull/3504)