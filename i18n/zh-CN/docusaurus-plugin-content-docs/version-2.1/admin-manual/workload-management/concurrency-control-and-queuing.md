---
{
"title": "并发控制与排队",
"language": "zh-CN"
}
---

并发控制与排队是一种资源管理机制，当多个查询同时请求资源，达到系统并发能力的上限时，Doris 会根据预设的策略和限制条件对查询进行排队管理，确保系统在高负载情况下仍能平稳运行，避免 OOM、系统卡死等问题。

Doris 的并发控制与排队机制主要通过 workload group 来实现。workload group 定义了查询的资源使用上限，包括最大并发数、排队队列长度和超时时间等参数。通过合理配置这些参数，可以达到资源管控的目的。

## 基本使用

```
create workload group if not exists queue_group
properties (
    "max_concurrency" = "10",
    "max_queue_size" = "20",
    "queue_timeout" = "3000"
);
```

**参数说明**


| 属性名称                       | 数据类型  | 默认值 |  取值范围   | 说明                                                                                                                                                                                                                                                                                                                                                                       |
|------------------------------|---------|-----|-----|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| max_concurrency              | 整型      | 2147483647 | [0, 2147483647] | 可选，最大查询并发数，默认值为整型最大值，也就是不做并发的限制。运行中的查询数量达到最大并发时，新来的查询会进入排队的逻辑。                                                                                                                                                                                                                                                                                                           |
| max_queue_size               | 整型      | 0          | [0, 2147483647] | 可选，查询排队队列的长度，当排队队列已满时，新来的查询会被拒绝。默认值为 0，含义是不排队。                                                                                                                                                                                                                                                                                                       |
| queue_timeout                | 整型      | 0          | [0, 2147483647] | 可选，查询在排队队列中的最大等待时间，单位为毫秒。如果查询在队列中的排队时间超过这个值，那么就会直接抛出异常给客户端。默认值为 0，含义是不排队，查询进入队列后立即返回失败。                                                                                                                                                                                                                                                                                  |

如果集群中目前有 1 台 FE，那么这个配置的含义为，集群中同时运行的查询数最大不超过 10 个，当最大并发已满时，新来的查询会排队，队列的长度不超过 20。查询在队列中排队的时间最长为 3s，排队超过 3s 的查询会直接返回失败给客户端。

:::tip
目前的排队设计是不感知 FE 的个数的，排队的参数只在单 FE 粒度生效，例如：

一个 Doris 集群配置了一个 workload group，设置 max_concurrency = 1；
如果集群中有 1FE，那么这个 workload group 在 Doris 集群视角看同时只会运行一个 SQL；
如果有 3 台 FE，那么在 Doris 集群视角看最大可运行的 SQL 个数为 3。
:::

## 查看排队状态

**语法**

```
show workload groups
```

**示例**

```
mysql [(none)]>show workload groups\G;
*************************** 1. row ***************************
                          Id: 1
                        Name: normal
                   cpu_share: 20
                memory_limit: 50%
    enable_memory_overcommit: true
             max_concurrency: 2147483647
              max_queue_size: 0
               queue_timeout: 0
              cpu_hard_limit: 1%
             scan_thread_num: 16
  max_remote_scan_thread_num: -1
  min_remote_scan_thread_num: -1
        memory_low_watermark: 50%
       memory_high_watermark: 80%
                         tag: 
       read_bytes_per_second: -1
remote_read_bytes_per_second: -1
           running_query_num: 0
           waiting_query_num: 0
```

```running_query_num```代表运行中的查询数量，```waiting_query_num```代表排队中的查询数量

## 绕开排队的逻辑

在有些运维情况下，管理员账户需要绕开排队的逻辑执行 SQL 对系统进行一些管理操作，那么可以通过设置 session 变量，来绕开排队的逻辑：

```
set bypass_workload_group = true;
```
