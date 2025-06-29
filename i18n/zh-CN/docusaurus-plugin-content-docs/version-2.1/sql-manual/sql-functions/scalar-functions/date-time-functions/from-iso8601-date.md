---
{
    "title": "FROM_ISO8601_DATE",
    "language": "zh-CN"
}
---
## 描述

将 ISO8601 格式的日期表达式转化为 date 类型的日期表达式。

## 语法

```sql
from_iso8601_date(<dt>)
```

## 参数

| 参数 | 说明 |
| -- | -- |
| `<dt>` | ISO8601 格式的日期 |

## 返回值

 date 类型的日期表达式。

## 举例

```sql
SELECT from_iso8601_date('0000-01'),from_iso8601_date('0000-W01'),from_iso8601_date('0000-059');
```

```text
+------------------------------+-------------------------------+-------------------------------+
| from_iso8601_date('0000-01') | from_iso8601_date('0000-W01') | from_iso8601_date('0000-059') |
+------------------------------+-------------------------------+-------------------------------+
| 0000-01-01                   | 0000-01-03                    | 0000-02-28                    |
+------------------------------+-------------------------------+-------------------------------+
```