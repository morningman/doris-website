---
{
    "title": "ADMIN-SHOW-REPLICA-STATUS",
    "language": "zh-CN"
}
---

## ADMIN-SHOW-REPLICA-STATUS

### Name

ADMIN SHOW REPLICA STATUS

## 描述

该语句用于展示一个表或分区的副本状态信息。

语法：

```sql
 ADMIN SHOW REPLICA STATUS FROM [db_name.]tbl_name [PARTITION (p1, ...)]
[where_clause];
```

说明

1. where_clause:
               WHERE STATUS [!]= "replica_status"

2. replica_status:
            OK:                         replica 处于健康状态
            DEAD:                     replica 所在 Backend 不可用
            VERSION_ERROR:  replica 数据版本有缺失
            SCHEMA_ERROR:   replica 的 schema hash 不正确
            MISSING:                 replica 不存在

## 举例

1. 查看表全部的副本状态

    ```sql
    ADMIN SHOW REPLICA STATUS FROM db1.tbl1;
    ```

2. 查看表某个分区状态为 VERSION_ERROR 的副本

    ```sql
    ADMIN SHOW REPLICA STATUS FROM tbl1 PARTITION (p1, p2)
    WHERE STATUS = "VERSION_ERROR";
    ```

3. 查看表所有状态不健康的副本

    ```sql
    ADMIN SHOW REPLICA STATUS FROM tbl1
    WHERE STATUS != "OK";
    ```

### Keywords

    ADMIN, SHOW, REPLICA, STATUS, ADMIN SHOW

### Best Practice

