---
{
    "title": "JSONB_TYPE",
    "language": "zh-CN"
}
---

## jsonb_type

## 描述

用来判断json_path指定的字段在JSONB数据中的类型，如果字段不存在返回NULL，如果存在返回下面的类型之一

- object
- array
- null
- bool
- int
- bigint
- double
- string

## 语法

```sql
STRING jsonb_type(JSONB j, VARCHAR json_path)
```

## 举例

参考 [jsonb tutorial](../../sql-reference/Data-Types/JSONB.md) 中的示例

### keywords

jsonb_type

