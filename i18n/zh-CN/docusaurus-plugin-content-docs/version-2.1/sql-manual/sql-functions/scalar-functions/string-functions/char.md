---
{
    "title": "CHAR",
    "language": "zh-CN"
}
---

## 描述

将每个参数解释为整数，并返回一个字符串，该字符串由这些整数的代码值给出的字符组成。特殊情况：

- 如果结果字符串对于给定字符集是非法的，相应的转换结果为 NULL 值。

- 大于 `255` 的参数将转换为多个结果字节。例如，`char(15049882)`等价于`char(229, 164, 154)`。


## 语法

```sql
CHAR ( <expr> [ , <expr> ... ] [ USING <charset_name> ] )
```

## 参数

| 参数               | 说明                  |
|------------------|---------------------|
| `<expr>`         | 需要被计算为字符的整数         |
| `<charset_name>` | 返回值的编码，目前只支持 `utf8` |

## 返回值

参数列表 `<expr>` 对应字符组成的字符串。特殊情况：

- 如果结果字符串对于给定字符集是非法的，相应的转换结果为 NULL 值。

- 大于 `255` 的参数将转换为多个结果字节。例如，`CHAR(15049882)`等价于`CHAR(229, 164, 154)`。

## 举例

```sql
SELECT CHAR(68, 111, 114, 105, 115),CHAR(15049882, 15179199, 14989469),CHAR(255)
```

```text
+--------------------------------------+--------------------------------------------+-------------------+
| char('utf8', 68, 111, 114, 105, 115) | char('utf8', 15049882, 15179199, 14989469) | char('utf8', 255) |
+--------------------------------------+--------------------------------------------+-------------------+
| Doris                                | 多睿丝                                     | NULL              |
+--------------------------------------+--------------------------------------------+-------------------+
```