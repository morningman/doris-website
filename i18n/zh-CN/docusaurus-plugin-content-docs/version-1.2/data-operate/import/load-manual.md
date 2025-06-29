---
{
    "title": "导入总览",
    "language": "zh-CN"
}
---

# 导入总览

## 支持的数据源

Doris 提供多种数据导入方案，可以针对不同的数据源进行选择不同的数据导入方式。

### 按场景划分

| 数据源                               | 导入方式                                                     |
|-----------------------------------|----------------------------------------------------------|
| 对象存储（S3）,HDFS                     | [使用 Broker 导入数据](./import-scenes/external-storage-load.md) |
| 本地文件                              | [导入本地数据](./import-scenes/local-file-load.md)             |
| Kafka                             | [订阅 Kafka 数据](./import-scenes/kafka-load.md)               |
| Mysql,PostgreSQL,Oracle,SQLServer | [通过外部表同步数据](./import-scenes/external-table-load.md)      |
| 通过 JDBC 导入                          | [使用 JDBC 同步数据](./import-scenes/jdbc-load.md)               |
| 导入 JSON 格式数据                        | [JSON 格式数据导入](./import-way/load-json-format.md)           |
| MySQL Binlog                      | [Binlog Load](./import-way/binlog-load-manual.md)        |
| AutoMQ                            | [AutoMQ Load](../../ecosystem/automq-load.md)            |

### 按导入方式划分

| 导入方式名称 | 使用方式                                                     |
| ------------ | ------------------------------------------------------------ |
| Spark Load   | [通过 Spark 导入外部数据](./import-way/spark-load-manual.md) |
| Broker Load  | [通过 Broker 导入外部存储数据](./import-way/broker-load-manual.md) |
| Stream Load  | [流式导入数据 (本地文件及内存数据)](./import-way/stream-load-manual.md) |
| Routine Load | [导入 Kafka 数据](./import-way/routine-load-manual.md)       |
| Binlog Load  | [采集 MySQL Binlog 导入数据](./import-way/binlog-load-manual.md) |
| Insert Into  | [外部表通过 INSERT 方式导入数据](./import-way/insert-into-manual.md) |
| S3 Load      | [S3 协议的对象存储数据导入](./import-way/s3-load-manual.md) |

## 支持的数据格式

不同的导入方式支持的数据格式略有不同。

| 导入方式     | 支持的格式                |
| ------------ | ----------------------- |
| Broker Load  | parquet、orc、csv、gzip |
| Stream Load  | csv、json、parquet、orc |
| Routine Load | csv、json               |
| MySQL Load   | csv                    |

## 导入说明

Apache Doris 的数据导入实现有以下共性特征，这里分别介绍，以帮助大家更好的使用数据导入功能

## 导入的原子性保证

Doris 的每一个导入作业，不论是使用 Broker Load 进行批量导入，还是使用 INSERT 语句进行单条导入，都是一个完整的事务操作。导入事务可以保证一批次内的数据原子生效，不会出现部分数据写入的情况。

同时，一个导入作业都会有一个 Label。这个 Label 是在一个数据库（Database）下唯一的，用于唯一标识一个导入作业。Label 可以由用户指定，部分导入功能也会由系统自动生成。

Label 是用于保证对应的导入作业，仅能成功导入一次。一个被成功导入的 Label，再次使用时，会被拒绝并报错 `Label already used`。通过这个机制，可以在 Doris 侧做到 `At-Most-Once` 语义。如果结合上游系统的 `At-Least-Once` 语义，则可以实现导入数据的 `Exactly-Once` 语义。

关于原子性保证的最佳实践，可以参阅 导入事务和原子性。

## 同步及异步导入

导入方式分为同步和异步。对于同步导入方式，返回结果即表示导入成功还是失败。而对于异步导入方式，返回成功仅代表作业提交成功，不代表数据导入成功，需要使用对应的命令查看导入作业的运行状态。

## 导入 Array 类型

向量化场景才能支持 Array 函数，非向量化场景不支持。

如果想要应用 Array 函数导入数据，则应先启用向量化功能；然后需要根据 Array 函数的参数类型将输入参数列转换为 Array 类型；最后，就可以继续使用 Array 函数了。

例如以下导入，需要先将列 b14 和列 a13 先 cast 成`array<string>`类型，再运用`array_union`函数。

```sql
LOAD LABEL label_03_14_49_34_898986_19090452100 ( 
  DATA INFILE("hdfs://test.hdfs.com:9000/user/test/data/sys/load/array_test.data") 
  INTO TABLE `test_array_table` 
  COLUMNS TERMINATED BY "|" (`k1`, `a1`, `a2`, `a3`, `a4`, `a5`, `a6`, `a7`, `a8`, `a9`, `a10`, `a11`, `a12`, `a13`, `b14`) 
  SET(a14=array_union(cast(b14 as array<string>), cast(a13 as array<string>))) WHERE size(a2) > 270) 
  WITH BROKER "hdfs" ("username"="test_array", "password"="") 
  PROPERTIES( "max_filter_ratio"="0.8" );
```
