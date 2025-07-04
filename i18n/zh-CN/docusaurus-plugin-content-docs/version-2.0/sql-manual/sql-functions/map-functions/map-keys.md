---
{
    "title": "MAP_KEYS",
    "language": "zh-CN"
}
---

## 描述

## 语法

`ARRAY<K> map_keys(Map<K, V> map)`

将给定 `map` 的键提取成一个对应类型的 `ARRAY`

## 举例

```sql
mysql> select map_keys(map(1, "100", 0.1, 2));
+-------------------------------------------------------------------------------------------------+
| map_keys(map(cast(1 as DECIMALV3(2, 1)), '100', cast(0.1 as DECIMALV3(2, 1)), cast(2 as TEXT))) |
+-------------------------------------------------------------------------------------------------+
| [1.0, 0.1]                                                                                      |
+-------------------------------------------------------------------------------------------------+
1 row in set (0.15 sec)

mysql> select map_keys(map());
+-----------------+
| map_keys(map()) |
+-----------------+
| []              |
+-----------------+
1 row in set (0.12 sec)
```

### Keywords

MAP, KEYS, MAP_KEYS
