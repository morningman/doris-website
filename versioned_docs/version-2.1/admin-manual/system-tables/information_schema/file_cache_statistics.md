---
{
    "title": "file_cache_statistics",
    "language": "en"
}
---

## Overview

Used to view the metric information related to data cache on each BE node. The metric information is sourced from the monitoring metrics related to BE's data cache.

:::tip Tip
This system table is supported from versions 2.1.6 and 3.0.2.
:::

## Database

`information_schema`

## Table Information

| Column Name | Type | Description |
|---|---|---|
| BE_ID | BIGINT | BE node ID |
| BE_IP | VARCHAR(256) | BE node IP |
| CACHE_PATH | VARCHAR(256) | BE node cache path |
| METRIC_NAME | VARCHAR(256) | Metric name |
| METRIC_VALUE | DOUBLE | Metric value |

:::info Note

Doris different version may have different metrics

:::

### 2.1.x Metrics

> Only important metrics are listed.

- `normal_queue_curr_elements`

    Number of File Blocks currently in the cache.

- `normal_queue_max_elements`

    Maximum number of File Blocks allowed in the cache.

- `normal_queue_curr_size`

    Current cache size.

- `normal_queue_max_size`

    Maximum cache size allowed.

- `hits_ratio`

    Overall cache hit ratio since BE startup. Range 0-1.

- `hits_ratio_5m`

    Cache hit ratio in the last 5 minutes. Range 0-1.

- `hits_ratio_1h`

    Cache hit ratio in the last 1 hour. Range 0-1.

### 3.0.x Metrics

TODO

## Examples

1. Query all cache metrics

    ```sql
    mysql> select * from information_schema.file_cache_statistics;
    +-------+---------------+----------------------------+----------------------------+--------------------+
    | BE_ID | BE_IP         | CACHE_PATH                 | METRIC_NAME                | METRIC_VALUE       |
    +-------+---------------+----------------------------+----------------------------+--------------------+
    | 10003 | 172.20.32.136 | /mnt/output/be/file_cache/ | normal_queue_curr_elements |               1392 |
    | 10003 | 172.20.32.136 | /mnt/output/be/file_cache/ | normal_queue_curr_size     |          248922234 |
    | 10003 | 172.20.32.136 | /mnt/output/be/file_cache/ | normal_queue_max_elements  |             102400 |
    | 10003 | 172.20.32.136 | /mnt/output/be/file_cache/ | normal_queue_max_size      |        21474836480 |
    | 10003 | 172.20.32.136 | /mnt/output/be/file_cache/ | hits_ratio                 | 0.8539634687001242 |
    | 10003 | 172.20.32.136 | /mnt/output/be/file_cache/ | hits_ratio_1h              |                  0 |
    | 10003 | 172.20.32.136 | /mnt/output/be/file_cache/ | hits_ratio_5m              |                  0 |
    +-------+---------------+----------------------------+----------------------------+--------------------+
    ```

2. Query cache hit ratio and sort by hit ratio

    ```sql
    select * from information_schema.file_cache_statistics where METRIC_NAME = "hits_ratio" order by METRIC_VALUE desc;
    ```
