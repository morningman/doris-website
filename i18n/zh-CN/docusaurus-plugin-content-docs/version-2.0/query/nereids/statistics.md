---
{
"title": "统计信息",
"language": "zh-CN"
}
---

通过收集统计信息有助于优化器了解数据分布特性，在进行 CBO（基于成本优化）时优化器会利用这些统计信息来计算谓词的选择性，并估算每个执行计划的成本。从而选择更优的计划以大幅提升查询效率。

当前收集列的如下信息：

| 信息            | 描述                       |
| :-------------- | :------------------------- |
| `row_count`     | 总行数                 |
| `data_size`     | 总数据量    |
| `avg_size_byte` | 值的平均⻓度 |
| `ndv`           | 不同值数量      |
| `min`           | 最小值                   |
| `max`           | 最⼤值                   |
| `null_count`    | 空值数量               |




## 1. 收集统计信息

---

### 1.1 使用 ANALYZE 语句手动收集

Doris 支持用户通过提交 ANALYZE 语句来手动触发统计信息的收集和更新。

语法：

```SQL
ANALYZE < TABLE | DATABASE table_name | db_name > 
    [ (column_name [, ...]) ]
    [ [ WITH SYNC ] [ WITH SAMPLE PERCENT | ROWS ] ];
```

其中：

- table_name: 指定的目标表。可以是  `db_name.table_name`  形式。

- column_name: 指定的目标列。必须是  `table_name`  中存在的列，多个列名称用逗号分隔。

- sync：同步收集统计信息。收集完后返回。若不指定则异步执行并返回 JOB ID。

- sample percent | rows：抽样收集统计信息。可以指定抽样比例或者抽样行数。

默认情况下（不指定 WITH SAMPLE)，会对一张表全量采样。对于比较大的表（5GiB 以上），从集群资源的角度出发，一般情况下我们建议采样收集，采样的行数建议不低于 400 万行。下面是一些例子

对一张表全量收集统计信息：

```sql
ANALYZE TABLE lineitem;
```


对一张表按照 10% 的比例采样收集统计数据：

```sql
ANALYZE TABLE lineitem WITH SAMPLE PERCENT 10;
```

对一张表按采样 10 万行收集统计数据

```sql
ANALYZE TABLE lineitem WITH SAMPLE ROWS 100000;
```



### 1.2 自动收集

此功能从 2.0.3 开始正式支持，默认为全天开启状态。下面对其基本运行逻辑进行阐述，在每次导入事务提交后，Doris 将记录本次导入事务更新的表行数用以估算当前已有表的统计数据的健康度（对于没有收集过统计数据的表，其健康度为 0）。当表的健康度低于 60（可通过参数`table_stats_health_threshold`调节）时，Doris 会认为该表的统计信息已经过时，并在之后触发对该表的统计信息收集作业。而对于统计信息健康度高于 60 的表，则不会重复进行收集。

统计信息的收集作业本身需要占用一定的系统资源，为了尽可能降低开销，Doris 会使用采样的方式去收集，自动采样默认采样 4194304(2^22) 行，以尽可能降低对系统造成的负担并尽快完成收集作业。如果希望采样更多的行以获得更准确的数据分布信息，可通过调整参数`huge_table_default_sample_rows`增大采样行数。用户还可通过参数控制小表全量收集，大表收集时间间隔等行为。详细配置请参考详[3.1](statistics.md#31-会话变量)。

如果担心自动收集作业对业务造成干扰，可结合自身需求通过设置参数`auto_analyze_start_time`和参数`auto_analyze_end_time`指定自动收集作业在业务负载较低的时间段执行。也可以通过设置参数`enable_auto_analyze` 为`false`来彻底关闭本功能。

External catalog 默认不参与自动收集。因为 external catalog 往往包含海量历史数据，如果参与自动收集，可能占用过多资源。可以通过设置 catalog 的 property 来打开和关闭 external catalog 的自动收集。

```sql
ALTER CATALOG external_catalog SET PROPERTIES ('enable.auto.analyze'='true'); // 打开自动收集
ALTER CATALOG external_catalog SET PROPERTIES ('enable.auto.analyze'='false'); // 关闭自动收集
```



## 2. 作业管理



### 2.1 查看统计作业

通过 `SHOW ANALYZE` 来查看统计信息收集作业的信息。

语法如下：

```SQL
SHOW [AUTO] ANALYZE < table_name | job_id >
    [ WHERE [ STATE = [ "PENDING" | "RUNNING" | "FINISHED" | "FAILED" ] ] ];
```

- AUTO：仅仅展示自动收集历史作业信息。需要注意的是默认只保存过去 20000 个执行完毕的自动收集作业的状态。

- table_name：表名，指定后可查看该表对应的统计作业信息。可以是  `db_name.table_name`  形式。不指定时返回所有统计作业信息。

- job_id：统计信息作业 ID，执行 `ANALYZE` 异步收集时得到。不指定 id 时此命令返回所有统计作业信息。

输出：

| 列名                   | 说明         |
| :--------------------- | :----------- |
| `job_id`               | 统计作业 ID  |
| `catalog_name`         | catalog 名称 |
| `db_name`              | 数据库名称   |
| `tbl_name`             | 表名称       |
| `col_name`             | 列名称列表       |
| `job_type`             | 作业类型     |
| `analysis_type`        | 统计类型     |
| `message`              | 作业信息     |
| `last_exec_time_in_ms` | 上次执行时间 |
| `state`                | 作业状态     |
| `schedule_type`        | 调度方式     |

下面是一个例子：

```sql
mysql> show analyze 245073\G;
*************************** 1. row ***************************
              job_id: 245073
        catalog_name: internal
             db_name: default_cluster:tpch
            tbl_name: lineitem
            col_name: [l_returnflag,l_receiptdate,l_tax,l_shipmode,l_suppkey,l_shipdate,l_commitdate,l_partkey,l_orderkey,l_quantity,l_linestatus,l_comment,l_extendedprice,l_linenumber,l_discount,l_shipinstruct]
            job_type: MANUAL
       analysis_type: FUNDAMENTALS
             message: 
last_exec_time_in_ms: 2023-11-07 11:00:52
               state: FINISHED
            progress: 16 Finished  |  0 Failed  |  0 In Progress  |  16 Total
       schedule_type: ONCE
```



### 2.2 查看每列统计信息收集情况

每个收集作业中可以包含一到多个任务，每个任务对应一列的收集。用户可通过如下命令查看具体每列的统计信息收集完成情况。

语法：

```sql
SHOW ANALYZE TASK STATUS [job_id]
```

下面是一个例子：

```sql
mysql> show analyze task status 20038 ;
+---------+----------+---------+----------------------+----------+
| task_id | col_name | message | last_exec_time_in_ms | state    |
+---------+----------+---------+----------------------+----------+
| 20039   | col4     |         | 2023-06-01 17:22:15  | FINISHED |
| 20040   | col2     |         | 2023-06-01 17:22:15  | FINISHED |
| 20041   | col3     |         | 2023-06-01 17:22:15  | FINISHED |
| 20042   | col1     |         | 2023-06-01 17:22:15  | FINISHED |
+---------+----------+---------+----------------------+----------+


```



### 2.3 查看列统计信息

通过 `SHOW COLUMN STATS` 来查看列的各项统计数据。

语法如下：

```sql
SHOW COLUMN [cached] STATS table_name [ (column_name [, ...]) ];
```

其中：

- cached: 展示当前 FE 内存缓存中的统计信息。

- table_name: 收集统计信息的目标表。可以是  `db_name.table_name`  形式。

- column_name: 指定的目标列，必须是  `table_name`  中存在的列，多个列名称用逗号分隔。

下面是一个例子：

```sql
mysql> show column stats lineitem(l_tax)\G;
*************************** 1. row ***************************
  column_name: l_tax
        count: 6001215.0
          ndv: 9.0
     num_null: 0.0
    data_size: 4.800972E7
avg_size_byte: 8.0
          min: 0.00
          max: 0.08
       method: FULL
         type: FUNDAMENTALS
      trigger: MANUAL
  query_times: 0
 updated_time: 2023-11-07 11:00:46

```



### 2.4 表收集概况

通过 `SHOW TABLE STATS` 查看表的统计信息收集概况。

语法如下：

```sql
SHOW TABLE STATS table_name;
```

其中：

- table_name: 目标表表名。可以是  `db_name.table_name`  形式。

输出：

| 列名                | 说明                   |
| :------------------ | :--------------------- |
|`updated_rows`|自上次 ANALYZE 以来该表的更新行数|
|`query_times`|保留列，后续版本用以记录该表查询次数|
|`row_count`| 行数（不反映命令执行时的准确行数）|
|`updated_time`| 上次更新时间|
|`columns`| 收集过统计信息的列|
|`trigger`|触发方式|

下面是一个例子：

```sql
mysql> show table stats lineitem \G;
*************************** 1. row ***************************
updated_rows: 0
 query_times: 0
   row_count: 6001215
updated_time: 2023-11-07
     columns: [l_returnflag, l_receiptdate, l_tax, l_shipmode, l_suppkey, l_shipdate, l_commitdate, l_partkey, l_orderkey, l_quantity, l_linestatus, l_comment, l_extendedprice, l_linenumber, l_discount, l_shipinstruct]
     trigger: MANUAL
```



### 2.5 终止统计作业

通过 `KILL ANALYZE` 来终止正在运行的统计作业。

语法如下：

```sql
KILL ANALYZE job_id;
```

其中：

- job_id：统计信息作业 ID。执行 `ANALYZE` 异步收集统计信息时所返回的值，也可以通过 `SHOW ANALYZE` 语句获取。

示例：

- 终止 ID 为 52357 的统计作业。

```sql
mysql> KILL ANALYZE 52357;
```



## 3. 会话变量及配置项



### 3.1 会话变量

|会话变量 | 说明 | 默认值|
|---|---|---|
|auto_analyze_start_time|自动统计信息收集开始时间|00:00:00|
|auto_analyze_end_time|自动统计信息收集结束时间|23:59:59|
|enable_auto_analyze|开启自动收集功能|true|
|huge_table_default_sample_rows|对大表的采样行数|4194304|
|huge_table_lower_bound_size_in_bytes|大小超过该值的的表，在自动收集时将会自动通过采样收集统计信息|0|
|huge_table_auto_analyze_interval_in_millis|控制对大表的自动 ANALYZE 的最小时间间隔，在该时间间隔内大小超过 huge_table_lower_bound_size_in_bytes * 5 的表仅 ANALYZE 一次|0|
|table_stats_health_threshold|取值在 0-100 之间，当自上次统计信息收集操作之后，数据更新量达到 (100 - table_stats_health_threshold)% ，认为该表的统计信息已过时|60|
|analyze_timeout|控制 ANALYZE 超时时间，单位为秒|43200|
|auto_analyze_table_width_threshold|控制自动统计信息收集处理的最大表宽度，列数大于该值的表不会参与自动统计信息收集|100|



### 3.2 FE 配置项

下面的 FE 配置项通常情况下，无需关注

|FE 配置项 | 说明 | 默认值|
|---|---|---|
|analyze_record_limit|控制统计信息作业执行记录的持久化行数|20000|
|stats_cache_size| FE 侧统计信息缓存条数 | 500000                        |
| statistics_simultaneously_running_task_num |可同时执行的异步作业数量|3|
| statistics_sql_mem_limit_in_bytes| 控制每个统计信息 SQL 可占用的 BE 内存 | 2L * 1024 * 1024 * 1024 (2GiB) |



## 4. 常见问题



### 4.1 ANALYZE 提交报错：Stats table not available...

执行 ANALYZE 时统计数据会被写入到内部表`__internal_schema.column_statistics`中，FE 会在执行 ANALYZE 前检查该表 tablet 状态，如果存在不可用的 tablet 则拒绝执行作业。出现该报错请检查 BE 集群状态。

用户可通过`SHOW BACKENDS\G`，确定 BE 状态是否正常。如果 BE 状态正常，可使用命令`ADMIN SHOW REPLICA STATUS FROM __internal_schema.[tbl_in_this_db]`，检查该库下 tablet 状态，确保 tablet 状态正常。



### 4.2 大表 ANALYZE 失败

由于 ANALYZE 能够使用的资源受到比较严格的限制，对一些大表的 ANALYZE 操作有可能超时或者超出 BE 内存限制。这些情况下，建议使用 `ANALYZE ... WITH SAMPLE...`。
