---
{
    "title": "ABS",
    "language": "zh-CN"
}
---

## 描述

返回一个值的绝对值。

## 语法

```sql
ABS(<x>)
```

## 参数

| 参数 | 说明 |
| -- | -- |
| `<x>` | 需要被计算绝对值的值 |

## 返回值

参数 x 的绝对值

## 举例

```sql
select abs(-2);
```

```text
+---------+
| abs(-2) |
+---------+
|       2 |
+---------+
```

```sql
select abs(3.254655654);
```

```text
+------------------+
| abs(3.254655654) |
+------------------+
|      3.254655654 |
+------------------+
```

```sql
select abs(-3254654236547654354654767);
```

```text
+---------------------------------+
| abs(-3254654236547654354654767) |
+---------------------------------+
| 3254654236547654354654767       |
+---------------------------------+
```
