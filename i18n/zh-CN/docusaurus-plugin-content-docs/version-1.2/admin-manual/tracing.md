---
{
    "title": "链路追踪",
    "language": "zh-CN"
}
---

链路追踪（tracing）记录了一次请求在系统中的执行的生命周期，包括请求及其子过程调用链路、执行时间及统计信息，可用于慢查询定位、性能瓶颈分析等。

:::caution 注意
该功能只维护于 Doris 1.2 版本，自 Doris 2.0 版本起已不再支持。
:::

## 原理

Doris 负责收集 traces，并导出到第三方链路分析系统，由链路分析系统负责 traces 的展示和存储。

## 快速搭建
Doris 目前支持直接将 traces 导出到 [zipkin](https://zipkin.io/) 中。

### 部署 zipkin

```
curl -sSL https://zipkin.io/quickstart.sh | bash -s
java -jar zipkin.jar
```

### 配置及启动 Doris

#### 添加配置到 fe.conf

```
# 开启链路追踪
enable_tracing = true

# 配置traces导出到zipkin
trace_export_url = http://127.0.0.1:9411/api/v2/spans
```

#### 添加配置到 be.conf
```
# 开启链路追踪。
enable_tracing = true

# 配置traces导出到zipkin。
trace_export_url = http://127.0.0.1:9411/api/v2/spans

# 可选。缓存span的队列大小。span数量达到队列容量一半时将触发一次span导出，队列满后到达队列的span将被丢弃。
max_span_queue_size=2048

# 可选。单次导出span的最大数量。
max_span_export_batch_size=512

# 可选。导出span的最大间隔时间。
export_span_schedule_delay_millis=500
```

#### 启动 fe 和 be
```
sh fe/bin/start_fe.sh --daemon
sh be/bin/start_be.sh --daemon
```

### 执行查询
```
...
```

### 查看 zipkin UI

浏览器打开`http://127.0.0.1:9411/zipkin/` 可查看查询链路。

## 使用 opentelemetry collector

使用 opentelemetry collector 可将 traces 导出到其他系统例如 zipkin、jaeger、skywalking，或者数据库系统和文件中。详情参考 [collector exporter](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/exporter) 。

同时 opentelemetry collector 提供了丰富的算子用来处理 traces。例如[过滤 spans](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/processor/filterprocessor) 、[尾采样](hhttps://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/processor/tailsamplingprocessor)。详情参考[collector processor](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/processor)。

traces 导出的路径：doris -> collector -> zipkin 等。

### 部署 opentelemetry collector

opentelemetry 发布了 collector [core](https://github.com/open-telemetry/opentelemetry-collector) 和 [contrib](https://github.com/open-telemetry/opentelemetry-collector-contrib), contrib 提供了更丰富的功能，这里以 contrib 版举例。

#### 下载 collector

下载 otelcol-contrib，可在官网下载[更多平台预编译版](https://github.com/open-telemetry/opentelemetry-collector-releases/releases)

```
wget https://github.com/open-telemetry/opentelemetry-collector-releases/releases/download/v0.55.0/otelcol-contrib_0.55.0_linux_amd64.tar.gz

tar -zxvf otelcol-contrib_0.55.0_linux_amd64.tar.gz
```

#### 生成配置文件

collector 配置文件分为 5 部分：`receivers`、`processors`、`exporters`、`extensions`、`service`。其中 receivers、processors、exporters 分别定义了接收、处理、导出数据的方式；extensions 是可选的，用于扩展主要用于不涉及处理遥测数据的任务；service 指定在 collector 中使用哪些组件。可参考 [collector configuration](https://opentelemetry.io/docs/collector/deployment/)。

下面配置文件使用 otlp(OpenTelemetry Protocol) 协议接收 traces 数据，进行批处理并过滤掉时间超过 50ms 的 traces, 最终导出到 zipkin 和文件中。
```
cat > otel-collector-config.yaml << EOF
receivers:
  otlp:
    protocols:
      http:

exporters:
  zipkin:
    endpoint: "http://10.81.85.90:8791/api/v2/spans"
  file:
    path: ./filename.json

processors:
  batch:
  tail_sampling:
    policies:
      {
        name: duration_policy,
        type: latency,
        latency: {threshold_ms: 50}
      }

extensions:

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [batch, tail_sampling]
      exporters: [zipkin, file]
EOF
```

#### 启动 collector

```
nohup ./otelcol-contrib --config=otel-collector-config.yaml &
```

### 配置及启动 Doris

#### 添加配置到 fe.conf

```
# 开启链路追踪
enable_tracing = true

# 启用opentelemetry collector。
trace_exporter = collector

# 配置traces导出到collector，4318为collector otlp http默认端口。
trace_export_url = http://127.0.0.1:4318/v1/traces
```

#### 添加配置到 be.conf
```
# 开启链路追踪。
enable_tracing = true

# 启用opentelemetry collector。
trace_exporter = collector

# 配置traces导出到collector，4318为collector otlp http默认端口。
trace_export_url = http://127.0.0.1:4318/v1/traces

# 可选。缓存span的队列大小。span数量达到队列容量一半时将触发一次span导出，队列满后到达队列的span将被丢弃。
max_span_queue_size=2048

# 可选。单次导出span的最大数量。
max_span_export_batch_size=512

# 可选。导出span的最大间隔时间。
export_span_schedule_delay_millis=500
```

#### 启动 fe 和 be
```
sh fe/bin/start_fe.sh --daemon
sh be/bin/start_be.sh --daemon
```

### 执行查询
```
...
```
### 查看 traces
```
...
```
