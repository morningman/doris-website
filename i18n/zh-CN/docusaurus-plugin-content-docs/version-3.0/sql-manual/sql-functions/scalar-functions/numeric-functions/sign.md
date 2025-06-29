---
{
    "title": "SIGN",
    "language": "zh-CN"
}
---

## 描述

返回`x`的符号。负数，零或正数分别对应 -1，0 或 1。

## 语法

```sql
SIGN(x)
```

## 参数

| 参数 | 说明 |
| -- | -- |
| `<x>` | 自变量 |

## 返回值

返回一个整型：

- 当 x > 0 时，返回 1，代表整数。

- 当 x = 0 时，返回 0，代表零。

- 当 x < 0 时，返回 -1，代表负数。

- 当 x is NULL 时，返回 NULL。

## 举例

```sql
select sign(3);
```

```text
+-------------------------+
| sign(cast(3 as DOUBLE)) |
+-------------------------+
|                       1 |
+-------------------------+
```

```sql
select sign(0);
```

```text
+-------------------------+
| sign(cast(0 as DOUBLE)) |
+-------------------------+
|                       0 |
+-------------------------+
```

```sql
select sign(-10.0);
```

```text
+-----------------------------+
| sign(cast(-10.0 as DOUBLE)) |
+-----------------------------+
|                          -1 |
+-----------------------------+
```

```sql
select sign(null);
```

```text
+------------+
| sign(NULL) |
+------------+
|       NULL |
+------------+
```