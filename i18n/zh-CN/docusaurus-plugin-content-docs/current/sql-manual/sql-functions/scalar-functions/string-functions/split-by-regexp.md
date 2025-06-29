---
{
    "title": "SPLIT_BY_REGEXP",
    "language": "zh-CN"
}
---

## 描述

将输入字符串按照指定的正则表达式拆分成字符串数组。

## 语法

```sql
SPLIT_BY_REGEXP ( <str>, <pattern> [, <max_limit>] )
```

## 参数

| 参数           | 说明                           |
|--------------|------------------------------|
| `<str>`      | 需要分割的字符串                     |
| `<pattern>`  | 正则表达式                        |
| `<max_limit>` | 可选参数，是否限制返回的字符串数组元素个数，默认是不限制 |

## 返回值

返回按照指定的正则表达式拆分成字符串数组。特殊情况：

- 任意参数中有一个为 NULL，则返回 NULL

## 举例

```sql
SELECT split_by_regexp('abcde',"");
```

```text
+------------------------------+
| split_by_regexp('abcde', '') |
+------------------------------+
| ["a", "b", "c", "d", "e"]    |
+------------------------------+
```

```sql
select split_by_regexp('a12bc23de345f',"\\d+");
```

```text
+-----------------------------------------+
| split_by_regexp('a12bc23de345f', '\d+') |
+-----------------------------------------+
| ["a", "bc", "de", "f"]                  |
+-----------------------------------------+
```
