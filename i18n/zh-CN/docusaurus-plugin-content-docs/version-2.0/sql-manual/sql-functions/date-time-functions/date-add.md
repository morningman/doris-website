---
{
    "title": "DATE_ADD",
    "language": "zh-CN"
}
---

## date_add
## 描述
## 语法

`INT DATE_ADD(DATETIME date, INTERVAL expr type)`


向日期添加指定的时间间隔。

date 参数是合法的日期表达式。

expr 参数是您希望添加的时间间隔。

type 参数可以是下列值：YEAR, MONTH, DAY, HOUR, MINUTE, SECOND

## 举例

```
mysql> select date_add('2010-11-30 23:59:59', INTERVAL 2 DAY);
+-------------------------------------------------+
| date_add('2010-11-30 23:59:59', INTERVAL 2 DAY) |
+-------------------------------------------------+
| 2010-12-02 23:59:59                             |
+-------------------------------------------------+
```

### keywords

    DATE_ADD,DATE,ADD
