# **第七节 为Go应用添加Prometheus监控指标**


## **1、创建应用**

我们首先从一个最简单的 Go 应用程序开始，在端口 `8080` 的 `/metrics` 端点上暴露客户端库的默认注册表，暂时还没有跟踪任何其他自定义的监控指标。

先创建一个名为 `instrument-demo` 的目录，在该目录下面初始化项目：

```
☸ ➜ mkdir instrument-demo && cd instrument-demo
☸ ➜ go mod init github.com/cnych/instrument-demo
```

**上面的命令会在 `instrument-demo` 目录下面生成一个 `go.mod` 文件，在同目录下面新建一个 `main.go` 的入口文件，内容如下所示**：

```
package main

import (
 "net/http"

 "github.com/prometheus/client_golang/prometheus/promhttp"
)

func main() {
    // Serve the default Prometheus metrics registry over HTTP on /metrics.
 http.Handle("/metrics", promhttp.Handler())
 http.ListenAndServe(":8080", nil)
}
```

然后执行下面的命令下载 Prometheus 客户端库依赖：

```
☸ ➜ export GOPROXY="https://goproxy.cn"
☸ ➜ go mod tidy
go: finding module for package github.com/prometheus/client_golang/prometheus/promhttp
go: found github.com/prometheus/client_golang/prometheus/promhttp in github.com/prometheus/client_golang v1.11.0
go: downloading google.golang.org/protobuf v1.26.0-rc.1
```


然后直接执行 `go run` 命令启动服务：

```
☸ ➜ go run main.go
```

然后我们可以在浏览器中访问 `http://localhost:8080/metrics` 来获得默认的监控指标数据：

```
# HELP go_gc_duration_seconds A summary of the pause duration of garbage collection cycles.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 0
go_gc_duration_seconds{quantile="0.25"} 0
go_gc_duration_seconds{quantile="0.5"} 0
go_gc_duration_seconds{quantile="0.75"} 0
go_gc_duration_seconds{quantile="1"} 0
go_gc_duration_seconds_sum 0
go_gc_duration_seconds_count 0
# HELP go_goroutines Number of goroutines that currently exist.
# TYPE go_goroutines gauge
go_goroutines 6
......
# HELP go_threads Number of OS threads created.
# TYPE go_threads gauge
go_threads 8
# HELP promhttp_metric_handler_requests_in_flight Current number of scrapes being served.
# TYPE promhttp_metric_handler_requests_in_flight gauge
promhttp_metric_handler_requests_in_flight 1
# HELP promhttp_metric_handler_requests_total Total number of scrapes by HTTP status code.
# TYPE promhttp_metric_handler_requests_total counter
promhttp_metric_handler_requests_total{code="200"} 1
promhttp_metric_handler_requests_total{code="500"} 0
promhttp_metric_handler_requests_total{code="503"} 0
```

我们并没有在代码中添加什么业务逻辑，但是可以看到依然有一些指标数据输出，这是因为 Go 客户端库默认在我们暴露的全局默认指标注册表中注册了一些关于 `promhttp` 处理器和运行时间相关的默认指标，根据不同指标名称的前缀可以看出：

* `go_*`：以 `go_` 为前缀的指标是关于 `Go` 运行时相关的指标，比如垃圾回收时间、`goroutine `数量等，这些都是 Go 客户端库特有的，其他语言的客户端库可能会暴露各自语言的其他运行时指标。
* **`promhttp_*`：来自 `promhttp` 工具包的相关指标，用于跟踪对指标请求的处理**。

这些默认的指标是非常有用，但是更多的时候我们需要自己控制，来暴露一些自定义指标。这就需要我们去实现自定义的指标了。

## **2、添加自定义指标**

接下来我们来自定义一个的 gauge 指标来暴露当前的温度。创建一个新的文件 `custom-metric/main.go`，内容如下所示：

```
package main

import (
 "net/http"

 "github.com/prometheus/client_golang/prometheus"
 "github.com/prometheus/client_golang/prometheus/promhttp"
)

func main() {
    // 创建一个没有任何 label 标签的 gauge 指标
 temp := prometheus.NewGauge(prometheus.GaugeOpts{
  Name: "home_temperature_celsius",
  Help: "The current temperature in degrees Celsius.",
 })

 // 在默认的注册表中注册该指标
 prometheus.MustRegister(temp)

 // 设置 gauge 的值为 39
 temp.Set(39)

 // 暴露指标
 http.Handle("/metrics", promhttp.Handler())
 http.ListenAndServe(":8080", nil)
}
```
上面文件中和最初的文件就有一些变化了：


* 我们使用 `prometheus.NewGauge()` 函数创建了一个自定义的 `gauge` 指标对象，指标名称为 `home_temperature_celsius`，并添加了一个注释信息。
* 然后使用 `prometheus.MustRegister() ` 函数在默认的注册表中注册了这个 `gauge` 指标对象。
* 通过调用 `Set()` 方法将 gauge 指标的值设置为 39。
* 然后像之前一样通过 `HTTP `暴露默认的注册表。

需要注意的是除了 `prometheus.MustRegister()` 函数之外还有一个 `prometheus.Register()` 函数，一般在 golang 中我们会将 `Mustxxx` 开头的函数定义为必须满足条件的函数，如果不满足会返回一个 panic 而不是一个 error 操作，所以如果这里不能正常注册的话会抛出一个 panic。


现在我们来运行这个程序：

```
☸ ➜ go run ./custom-metric
```

启动后重新访问指标接口 `http://localhost:8080/metrics`，仔细对比我们会发现多了一个名为 `home_temperature_celsius` 的指标：

```
...
# HELP home_temperature_celsius The current temperature in degrees Celsius.
# TYPE home_temperature_celsius gauge
home_temperature_celsius 42
...
```

这样我们就实现了添加一个自定义的指标的操作，整体比较简单，当然在实际的项目中需要结合业务来确定添加哪些自定义指标。


## **3、自定义注册表**

### **3-1 Gauges**

前面的示例我们已经了解了如何添加 gauge 类型的指标，创建了一个没有任何标签的指标，直接使用 `prometheus.NewGauge()` 函数即可实例化一个 gauge 类型的指标对象，通过 `prometheus.GaugeOpts` 对象可以指定指标的名称和注释信息：

```
queueLength := prometheus.NewGauge(prometheus.GaugeOpts{
 Name: "queue_length",
 Help: "The number of items in the queue.",
})
```

**我们知道 gauge 类型的指标值是可以上升或下降的，所以我们可以为 gauge 指标设置一个指定的值，所以 gauge 指标对象暴露了 `Set()`、`Inc()`、`Dec()`、`Add()` 和 `Sub() `这些函数来更改指标值**：


```
// 使用 Set() 设置指定的值
queueLength.Set(0)

// 增加或减少
queueLength.Inc()   // +1：Increment the gauge by 1.
queueLength.Dec()   // -1：Decrement the gauge by 1.
queueLength.Add(23) // Increment by 23.
queueLength.Sub(42) // Decrement by 42.
```

另外 gauge 仪表盘经常被用来暴露 Unix 的时间戳样本值，所以也有一个方便的方法来将 gauge 设置为当前的时间戳：

```
demoTimestamp.SetToCurrentTime()
```

最终 gauge 指标会被渲染成如下所示的数据：

```

# HELP queue_length The number of items in the queue.
# TYPE queue_length gauge
queue_length 42
```

### **3-2 Counters**

要创建一个 counter 类型的指标和 gauge 比较类似，只是用` prometheus.NewCounter() `函数来初始化指标对象：

```
totalRequests := prometheus.NewCounter(prometheus.CounterOpts{
 Name: "http_requests_total",
 Help: "The total number of handled HTTP requests.",
})
```

我们知道 counter 指标只能随着时间的推移而不断增加，所以我们不能为其设置一个指定的值或者减少指标值，所以该对象下面只有 `Inc()` 和 `Add()` 两个函数：

```
totalRequests.Inc()   // +1：Increment the counter by 1.
totalRequests.Add(23) // +n：Increment the counter by 23.
```

当服务进程重新启动的时候，counter 指标值会被重置为 0，不过不用担心数据错乱，我们一般会使用的 `rate() `函数会自动处理。


```
# HELP http_requests_total The total number of handled HTTP requests.
# TYPE http_requests_total counter
http_requests_total 7734
```


### **3-3 Histograms**

创建直方图指标比 counter 和 gauge 都要复杂，因为需要配置把观测值归入的 bucket 的数量，以及每个 bucket 的上边界。

Prometheus 中的直方图是累积的，所以每一个后续的 bucket 都包含前一个 bucket 的观察计数，所有 bucket 的下限都从 0 开始的，所以我们不需要明确配置每个 bucket 的下限，只需要配置上限即可。

同样要创建直方图指标对象，我们使用 `prometheus.NewHistogram() `函数来进行初始化：

```
requestDurations := prometheus.NewHistogram(prometheus.HistogramOpts{
  Name:    "http_request_duration_seconds",
  Help:    "A histogram of the HTTP request durations in seconds.",
  // Bucket 配置：第一个 bucket 包括所有在 0.05s 内完成的请求，最后一个包括所有在10s内完成的请求。
  Buckets: []float64{0.05, 0.1, 0.25, 0.5, 1, 2.5, 5, 10},
})
```

这里和前面不一样的地方在于除了指定指标名称和帮助信息之外，还需要配置 Buckets。如果我们手动去枚举所有的 bucket 可能很繁琐，所以 Go 客户端库为为我们提供了一些辅助函数可以帮助我们生成线性或者指数增长的 bucket，比如 `prometheus.LinearBuckets()` 和 `prometheus.ExponentialBuckets() `函数。

直方图会自动对数值的分布进行分类和计数，所以它只有一个 `Observe() `方法，每当你在代码中处理要跟踪的数据时，就会调用这个方法。例如，如果你刚刚处理了一个 `HTTP` 请求，花了 `0.42` 秒，则可以使用下面的代码来跟踪。

```
requestDurations.Observe(0.42)
```

由于跟踪持续时间是直方图的一个常见用例，Go 客户端库就提供了辅助函数，用于对代码的某些部分进行计时，然后自动观察所产生的持续时间，将其转化为直方图，如下代码所示：

```

// 启动一个计时器
timer := prometheus.NewTimer(requestDurations)

// [...在应用中处理请求...]

// 停止计时器并观察其持续时间，将其放进 requestDurations 的直方图指标中去
timer.ObserveDuration()
```

直方图指标最终会生成如下所示的数据：

```
# HELP http_request_duration_seconds A histogram of the HTTP request durations in seconds.
# TYPE http_request_duration_seconds histogram
http_request_duration_seconds_bucket{le="0.05"} 4599
http_request_duration_seconds_bucket{le="0.1"} 24128
http_request_duration_seconds_bucket{le="0.25"} 45311
http_request_duration_seconds_bucket{le="0.5"} 59983
http_request_duration_seconds_bucket{le="1"} 60345
http_request_duration_seconds_bucket{le="2.5"} 114003
http_request_duration_seconds_bucket{le="5"} 201325
http_request_duration_seconds_bucket{le="+Inf"} 227420
http_request_duration_seconds_sum 88364.234
http_request_duration_seconds_count 227420
```

每个配置的存储桶最终作为一个带有 `_bucket` 后缀的计数器时间序列，使用 `le（小于或等于`） 标签指示该存储桶的上限，具有上限的隐式存储桶 `+Inf` 也暴露于比最大配置的存储桶边界花费更长的时间的请求，还包括使用后缀 `_sum `累积总和和计数 `_count` 的指标，这些时间序列中的每一个在概念上都是一个 `counter` 计数器（只能上升的单个值），只是它们是作为直方图的一部分创建的。

### **3-4 Summaries**

创建和使用摘要与直方图非常类似，只是我们需要指定要跟踪的 quantiles 分位数值，而不需要处理 bucket 桶，比如我们想要跟踪 HTTP 请求延迟的第 50、90 和 99 个百分位数，那么我们可以创建这样的一个摘要对象：

```
requestDurations := prometheus.NewSummary(prometheus.SummaryOpts{
    Name:       "http_request_duration_seconds",
    Help:       "A summary of the HTTP request durations in seconds.",
    Objectives: map[float64]float64{
      0.5: 0.05,   // 第50个百分位数，最大绝对误差为0.05。
      0.9: 0.01,   // 第90个百分位数，最大绝对误差为0.01。
      0.99: 0.001, // 第90个百分位数，最大绝对误差为0.001。
    },
  },
)
```

**这里和前面不一样的地方在于使用 `prometheus.NewSummary()` 函数初始化摘要指标对象的时候，需要通过 `prometheus.SummaryOpts{}` 对象的 `Objectives` 属性指定想要跟踪的分位数值**。

同样摘要指标对象创建后，跟踪持续时间的方式和直方图是完全一样的，使用一个 `Observe()` 函数即可：

```
requestDurations.Observe(0.42)
```

虽然直方图桶可以跨维度汇总（如端点、HTTP 方法等），但这对于汇总 quantiles 分位数值来说在统计学上是无效的。例如，你不能对两个单独的服务实例的第 90 百分位延迟进行平均，并期望得到一个有效的整体第 90 百分位延迟。如果需要按维度进行汇总，那么我们需要使用直方图而不是摘要指标。

摘要指标最终生成的指标数据与直方图非常类似，不同之处在于使用 `quantile` 标签来表示分位数序列，并且这些序列没有扩展指标名称的后缀：

```

# HELP http_request_duration_seconds A summary of the HTTP request durations in seconds.
# TYPE http_request_duration_seconds summary
http_request_duration_seconds{quantile="0.5"} 0.052
http_request_duration_seconds{quantile="0.90"} 0.564
http_request_duration_seconds{quantile="0.99"} 2.372
http_request_duration_seconds_sum 88364.234
http_request_duration_seconds_count 227420
```

### **3-5 标签**

到目前为止，我们还没有为指标对象添加任何的标签，要创建具有标签维度的指标，我们可以调用类似于 `NewXXXVec() `的构造函数来初始化指标对象：



* `NewGauge()` 变成 `NewGaugeVec()`
* `NewCounter() `变成 `NewCounterVec()`
* `NewSummary()` 变成 `NewSummaryVec()`
* `NewHistogram()` 变成 `NewHistogramVec()`

这些函数允许我们指定一个额外的字符串切片参数，提供标签名称的列表，通过它来拆分指标。

例如，为了按照房子以及测量温度的房间来划分我们早期的温度表指标，可以这样创建指标。

```
temp := prometheus.NewGaugeVec(
  prometheus.GaugeOpts{
    Name: "home_temperature_celsius",
    Help: "The current temperature in degrees Celsius.",
  },
  // 两个标签名称，通过它们来分割指标。
  []string{"house", "room"},
)
```

然后要访问一个特有标签的子指标，需要在设置其值之前，用 `house` 和 `room` 标签的各自数值，对产生的 `gauge` 向量调用 `WithLabelValues() `方法来处理下：

```
// 为 home=ydzs 和 room=living-room 设置指标值
temp.WithLabelValues("ydzs", "living-room").Set(27)
```

**如果你喜欢在选择的子指标中明确提供标签名称，可以使用效率稍低的 `With() `方法来代替**：

```
temp.With(prometheus.Labels{"house": "ydzs", "room": "living-room"}).Set(66)
```

不过需要注意如果向这两个方法传递不正确的标签数量或不正确的标签名称，这两个方法都会触发 panic。

下面是我们按照 `house` 和 `room` 标签维度区分指标的完整示例，创建一个名为 `label-metric/main.go `的新文件，内容如下所示：

```
package main

import (
 "net/http"

 "github.com/prometheus/client_golang/prometheus"
 "github.com/prometheus/client_golang/prometheus/promhttp"
)

func main() {
    // 创建带 house 和 room 标签的 gauge 指标对象
 temp := prometheus.NewGaugeVec(
  prometheus.GaugeOpts{
   Name: "home_temperature_celsius",
   Help: "The current temperature in degrees Celsius.",
  },
  // 指定标签名称
  []string{"house", "room"},
 )

 // 注册到全局默认注册表中
 prometheus.MustRegister(temp)

 // 针对不同标签值设置不同的指标值
 temp.WithLabelValues("cnych", "living-room").Set(27)
 temp.WithLabelValues("cnych", "bedroom").Set(25.3)
 temp.WithLabelValues("ydzs", "living-room").Set(24.5)
 temp.WithLabelValues("ydzs", "bedroom").Set(27.7)

 // 暴露自定义的指标
 http.Handle("/metrics", promhttp.Handler())
 http.ListenAndServe(":8080", nil)
}
```

上面代码非常清晰了，运行下面的程序：

```
☸ ➜ go run ./label-metric
```

启动完成后重新访问指标端点 `http://localhost:8080/metrics`，可以找到 `home_temperature_celsius` 指标不同标签维度下面的指标值：

```

...
# HELP home_temperature_celsius The current temperature in degrees Celsius.
# TYPE home_temperature_celsius gauge
home_temperature_celsius{house="cnych",room="bedroom"} 25.3
home_temperature_celsius{house="cnych",room="living-room"} 27
home_temperature_celsius{house="ydzs",room="bedroom"} 27.7
home_temperature_celsius{house="ydzs",room="living-room"} 24.5
...
```

> 注意：当使用带有标签维度的指标时，任何标签组合的时间序列只有在该标签组合被访问过至少一次后才会出现在 /metrics 输出中，这对我们在 PromQL 查询的时候会产生一些问题，因为它希望某些时间序列一直存在，我们可以在程序第一次启动时，将所有重要的标签组合预先初始化为默认值。







