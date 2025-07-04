---
{
    "title": "Kill Query",
    "language": "zh-CN"
}
---

## Kill 连接

每个 Doris 的连接都在一个单独的线程中运行。您可以使用 KILL processlist_id 语句终止线程。

线程进程列表标识符可以从 SHOW PROCESSLIST 输出的 Id 列查询 或者 SELECT CONNECTION_ID() 来查询当前 Connection ID。

语法：

```SQL
KILL [CONNECTION] processlist_id
```

## Kill 查询

除此之外，您还可以使用 processlist_id 或者 query_id 终止正在执行的查询命令

语法：

```SQL
KILL QUERY processlist_id | query_id
```

## 举例

**1. 查看当前连接的 Connection ID。**

```sql
mysql select connection_id();
+-----------------+
| connection_id() |
+-----------------+
| 48              |
+-----------------+
1 row in set (0.00 sec)
```

**2. 查看所有连接的 Connection ID。**

```sql
mysql SHOW PROCESSLIST;
+------------------+------+------+--------------------+---------------------+----------+---------+---------+------+-------+-----------------------------------+---------------------------------------------------------------------------------------+
| CurrentConnected | Id   | User | Host               | LoginTime           | Catalog  | Db      | Command | Time | State | QueryId                           | Info                                                                                  |
+------------------+------+------+--------------------+---------------------+----------+---------+---------+------+-------+-----------------------------------+---------------------------------------------------------------------------------------+
| Yes              |   48 | root | 10.16.xx.xx:44834   | 2023-12-29 16:49:47 | internal | test | Query   |    0 | OK    | e6e4ce9567b04859-8eeab8d6b5513e38 | SHOW PROCESSLIST                                                                      |
|                  |   50 | root | 192.168.xx.xx:52837 | 2023-12-29 16:51:34 | internal |      | Sleep   | 1837 | EOF   | deaf13c52b3b4a3b-b25e8254b50ff8cb | SELECT @@session.transaction_isolation                                                |
|                  |   51 | root | 192.168.xx.xx:52843 | 2023-12-29 16:51:35 | internal |      | Sleep   |  907 | EOF   | 437f219addc0404f-9befe7f6acf9a700 | /* ApplicationName=DBeaver Ultimate 23.1.3 - Metadata */ SHOW STATUS                  |
|                  |   55 | root | 192.168.xx.xx:55533 | 2023-12-29 17:09:32 | internal | test | Sleep   |  271 | EOF   | f02603dc163a4da3-beebbb5d1ced760c | /* ApplicationName=DBeaver Ultimate 23.1.3 - SQLEditor <Console> */ SELECT DATABASE() |
|                  |   47 | root | 10.16.xx.xx:35678   | 2023-12-29 16:21:56 | internal | test | Sleep   | 3528 | EOF   | f4944c543dc34a99-b0d0f3986c8f1c98 | select * from test                                                                    |
+------------------+------+------+--------------------+---------------------+----------+---------+---------+------+-------+-----------------------------------+---------------------------------------------------------------------------------------+
5 rows in set (0.00 sec)
```

**3. 终止正在运行的查询，正在运行的查询会显示被取消。**

```sql
mysql kill query 55;
Query OK, 0 rows affected (0.01 sec)
```