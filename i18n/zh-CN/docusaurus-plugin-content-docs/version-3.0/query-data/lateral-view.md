---
{
    "title": "列转行 (Lateral View)",
    "language": "zh-CN"
}
---

与生成器函数（例如 `EXPLODE`）结合使用，`LATERAL VIEW` 可以生成一个包含一个或多个行的虚拟表，并将这些行应用于每个原始输出行。  


## 语法

```sql
LATERAL VIEW generator_function ( expression [, ...] ) table_identifier AS column_identifier [, ...]
```


## 参数

- `generator_function`：生成器函数（如 EXPLODE、EXPLODE_SPLIT 等）。

- `table_identifier`：`generator_function` 的别名。

- `column_identifier`：列别名，用于输出行。列标识符的数量必须与生成器函数返回的列数匹配。

## 示例

假设有一个名为 `person` 的表，结构如下：


```sql
CREATE TABLE `person` (
  `id` int(11) NULL,
  `name` text NULL,
  `age` int(11) NULL,
  `class` int(11) NULL,
  `address` text NULL
) ENGINE=OLAP
UNIQUE KEY(`id`)
COMMENT 'OLAP'
DISTRIBUTED BY HASH(`id`) BUCKETS 1
PROPERTIES (
"replication_allocation" = "tag.location.default: 1",
"in_memory" = "false",
"storage_format" = "V2",
"disable_auto_compaction" = "false"
);

INSERT INTO person VALUES
    (100, 'John', 30, 1, 'Street 1'),
    (200, 'Mary', NULL, 1, 'Street 2'),
    (300, 'Mike', 80, 3, 'Street 3'),
    (400, 'Dan', 50, 4, 'Street 4');
```

使用 LATERAL VIEW 和 EXPLODE 函数查询 `person` 表：

```sql
SELECT * FROM person
LATERAL VIEW EXPLODE(ARRAY(30, 60)) tableName AS c_age;
```

查询结果将包含原始行的每个组合，以及 EXPLODE 函数生成的行：

```sql
+------+------+------+-------+----------+-------+
| id   | name | age  | class | address  | c_age |
+------+------+------+-------+----------+-------+
|  100 | John |   30 |     1 | Street 1 |    30 |
|  100 | John |   30 |     1 | Street 1 |    60 |
|  200 | Mary | NULL |     1 | Street 2 |    30 |
|  200 | Mary | NULL |     1 | Street 2 |    60 |
|  300 | Mike |   80 |     3 | Street 3 |    30 |
|  300 | Mike |   80 |     3 | Street 3 |    60 |
|  400 | Dan  |   50 |     4 | Street 4 |    30 |
|  400 | Dan  |   50 |     4 | Street 4 |    60 |
+------+------+------+-------+----------+-------+
8 rows in set (0.12 sec)
```

