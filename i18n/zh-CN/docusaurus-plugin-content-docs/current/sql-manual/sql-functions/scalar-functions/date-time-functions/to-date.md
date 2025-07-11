---
{
    "title": "TO_DATE",
    "language": "zh-CN"
}
---

## 描述 
日期转换函数，用于将日期时间（DATETIME）转换为日期类型（DATE），即去掉时间部分，仅保留日期（YYYY-MM-DD）

## 语法
```sql
TO_DATE(<datetime_value>)
```

## 必选参数
| 参数               | 描述                 |
|------------------|--------------------|
| `datetime_value` | DATETIME 类型日期时间    |


## 举例

将 `2020-02-02 00:00:00` 转换为 `2020-02-02`
```sql
select to_date("2020-02-02 00:00:00");
```
```text
+--------------------------------+
| to_date('2020-02-02 00:00:00') |
+--------------------------------+
| 2020-02-02                     |
+--------------------------------+
```