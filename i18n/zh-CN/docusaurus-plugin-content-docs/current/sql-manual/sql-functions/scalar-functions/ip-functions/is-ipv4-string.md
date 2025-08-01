---
{
    "title": "IS_IPV4_STRING",
    "language": "zh-CN"
}
---

## 描述
接收一个表示形式为字符串的 IPv4 地址作为参数，如果为格式正确且合法的 IPv4 地址，返回 true；反之，返回 false。

## 语法
```sql
IS_IPV4_STRING(<ipv4_str>)
```

## 参数
| Parameter | Description                                      |
|-----------|--------------------------------------------------|
| `<ipv4_str>`      | 字符串类型的 ipv4 地址 |

## 返回值
如果为格式正确且合法的 IPv4 地址，返回 true；反之，返回 false。
- 如果输入为 NULL, 则返回 NULL

## 举例
```sql
CREATE TABLE `test_is_ipv4_string` (
      `id` int,
      `ip_v4` string
    ) ENGINE=OLAP
    DISTRIBUTED BY HASH(`id`) BUCKETS 4
    PROPERTIES (
    "replication_allocation" = "tag.location.default: 1"
    );

insert into test_is_ipv4_string values(0, NULL), (1, '0.0.0.'), (2, ''), (3, '.'), (4, '255.255.255.255');

select id, ip_v4, is_ipv4_string(ip_v4) from test_is_ipv4_string order by id;
```
```text
+------+-----------------+-----------------------+
| id   | ip_v4           | is_ipv4_string(ip_v4) |
+------+-----------------+-----------------------+
|    0 | NULL            |                  NULL |
|    1 | 0.0.0.          |                     0 |
|    2 |                 |                     0 |
|    3 | .               |                     0 |
|    4 | 255.255.255.255 |                     1 |
+------+-----------------+-----------------------+
```
