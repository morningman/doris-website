---
{
    "title": "TRUNCATE-TABLE",
    "language": "zh-CN"
}
---

## TRUNCATE-TABLE

### Name

TRUNCATE TABLE

## 描述

该语句用于清空指定表和分区的数据
语法：

```sql
TRUNCATE TABLE [db.]tbl[ PARTITION(p1, p2, ...)];
```

说明：

- 该语句清空数据，但保留表或分区。
- 不同于 DELETE，该语句只能整体清空指定的表或分区，不能添加过滤条件。
- 不同于 DELETE，使用该方式清空数据不会对查询性能造成影响。
- 该操作删除的数据不可恢复。
- 使用该命令时，表状态需为 NORMAL，即不允许正在进行 SCHEMA CHANGE 等操作。
- 该命令可能会导致正在进行的导入失败。

## 举例

1. 清空 example_db 下的表 tbl

    ```sql
    TRUNCATE TABLE example_db.tbl;
    ```

2. 清空表 tbl 的 p1 和 p2 分区

    ```sql
    TRUNCATE TABLE tbl PARTITION(p1, p2);
    ```

### Keywords

    TRUNCATE, TABLE

### Best Practice

