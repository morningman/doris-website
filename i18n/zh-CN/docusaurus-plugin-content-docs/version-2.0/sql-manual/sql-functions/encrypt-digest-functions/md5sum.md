---
{
"title": "MD5SUM",
"language": "zh-CN"
}
---

### 描述

计算 多个字符串 MD5 128-bit

### 语法

```sql
MD5SUM(str[,str])
```

### 示例

```sql
select md5("abcd");
```

```
+----------------------------------+
| md5('abcd')                      |
+----------------------------------+
| e2fc714c4727ee9395f324cd2e7f331f |
+----------------------------------+
```

```sql
select md5sum("ab","cd");
```

```
+----------------------------------+
| md5sum('ab', 'cd')               |
+----------------------------------+
| e2fc714c4727ee9395f324cd2e7f331f |
+----------------------------------+
```
