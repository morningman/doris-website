---
{
    "title": "Querying Files on S3/HDFS",
    "language": "en"
}
---

With the Table Value Function feature, Doris is able to query files in object storage or HDFS as simply as querying Tables. In addition, it supports automatic column type inference.

## Usage

For more usage details, please see the documentation:

* [S3](../sql-manual/sql-functions/table-valued-functions/s3.md): supports file analysis on object storage compatible with S3

* [HDFS](../sql-manual/sql-functions/table-valued-functions/hdfs.md): supports file analysis on HDFS

* [LOCAL](../sql-manual/sql-functions/table-valued-functions/local.md): supports file analysis on local file system

The followings illustrate how file analysis is conducted with the example of S3 Table Value Function.

### Automatic Column Type Inference

```sql
> DESC FUNCTION s3 (
    "URI" = "http://127.0.0.1:9312/test2/test.snappy.parquet",
    "s3.access_key"= "ak",
    "s3.secret_key" = "sk",
    "format" = "parquet",
    "use_path_style"="true"
);
+---------------+--------------+------+-------+---------+-------+
| Field         | Type         | Null | Key   | Default | Extra |
+---------------+--------------+------+-------+---------+-------+
| p_partkey     | INT          | Yes  | false | NULL    | NONE  |
| p_name        | TEXT         | Yes  | false | NULL    | NONE  |
| p_mfgr        | TEXT         | Yes  | false | NULL    | NONE  |
| p_brand       | TEXT         | Yes  | false | NULL    | NONE  |
| p_type        | TEXT         | Yes  | false | NULL    | NONE  |
| p_size        | INT          | Yes  | false | NULL    | NONE  |
| p_container   | TEXT         | Yes  | false | NULL    | NONE  |
| p_retailprice | DECIMAL(9,0) | Yes  | false | NULL    | NONE  |
| p_comment     | TEXT         | Yes  | false | NULL    | NONE  |
+---------------+--------------+------+-------+---------+-------+
```

An S3 Table Value Function is defined as follows:

```sql
s3(
    "URI" = "http://127.0.0.1:9312/test2/test.snappy.parquet",
    "s3.access_key"= "ak",
    "s3.secret_key" = "sk",
    "Format" = "parquet",
    "use_path_style"="true")
```

It specifies the file path, connection, and authentication.

After defining, you can view the schema of this file using the `DESC FUNCTION` statement.

As can be seen, Doris is able to automatically infer column types based on the metadata of the Parquet file.

Besides Parquet, Doris supports analysis and auto column type inference of ORC, CSV, and Json files.

**CSV Schema**

By default, for CSV format files, all columns are of type String. Column names and column types can be specified individually via the `csv_schema` attribute. Doris will use the specified column type for file reading. The format is as follows:

`name1:type1;name2:type2;...`

For columns with mismatched formats (such as string in the file and int defined by the user), or missing columns (such as 4 columns in the file and 5 columns defined by the user), these columns will return null.

Currently supported column types are:

| name | mapping type |
| --- | --- |
|tinyint |tinyint |
|smallint |smallint |
|int |int |
| bigint | bigint |
| largeint | largeint |
| float| float |
| double| double|
| decimal(p,s) | decimalv3(p,s) |
| date | datev2 |
| datetime | datetimev2 |
| char |string |
|varchar |string |
|string|string |
|boolean| boolean |

Example:

```sql
s3 (
    "uri" = "https://bucket1/inventory.dat",
    "s3.access_key"= "ak",
    "s3.secret_key" = "sk",
    "format" = "csv",
    "column_separator" = "|",
    "csv_schema" = "k1:int;k2:int;k3:int;k4:decimal(38,10)",
    "use_path_style"="true"
)
```

### Query and Analysis

You can conduct queries and analysis on this Parquet file using any SQL statements:

```sql
SELECT * FROM s3(
    "uri" = "http://127.0.0.1:9312/test2/test.snappy.parquet",
    "s3.access_key"= "ak",
    "s3.secret_key" = "sk",
    "format" = "parquet",
    "use_path_style"="true")
LIMIT 5;
+-----------+------------------------------------------+----------------+----------+-------------------------+--------+-------------+---------------+---------------------+
| p_partkey | p_name                                   | p_mfgr         | p_brand  | p_type                  | p_size | p_container | p_retailprice | p_comment           |
+-----------+------------------------------------------+----------------+----------+-------------------------+--------+-------------+---------------+---------------------+
|         1 | goldenrod lavender spring chocolate lace | Manufacturer#1 | Brand#13 | PROMO BURNISHED COPPER  |      7 | JUMBO PKG   |           901 | ly. slyly ironi     |
|         2 | blush thistle blue yellow saddle         | Manufacturer#1 | Brand#13 | LARGE BRUSHED BRASS     |      1 | LG CASE     |           902 | lar accounts amo    |
|         3 | spring green yellow purple cornsilk      | Manufacturer#4 | Brand#42 | STANDARD POLISHED BRASS |     21 | WRAP CASE   |           903 | egular deposits hag |
|         4 | cornflower chocolate smoke green pink    | Manufacturer#3 | Brand#34 | SMALL PLATED BRASS      |     14 | MED DRUM    |           904 | p furiously r       |
|         5 | forest brown coral puff cream            | Manufacturer#3 | Brand#32 | STANDARD POLISHED TIN   |     15 | SM PKG      |           905 |  wake carefully     |
+-----------+------------------------------------------+----------------+----------+-------------------------+--------+-------------+---------------+---------------------+
```

You can put the Table Value Function anywhere that you used to put Table in the SQL, such as in the WITH or FROM clause in CTE. In this way, you can treat the file as a normal table and conduct analysis conveniently.

You can also create a logic view by using `CREATE VIEW` statement for a Table Value Function. So that you can query this view, grant priv on this view or allow other user to access this Table Value Function.

```sql
CREATE VIEW v1 AS 
SELECT * FROM s3(
    "uri" = "http://127.0.0.1:9312/test2/test.snappy.parquet",
    "s3.access_key"= "ak",
    "s3.secret_key" = "sk",
    "format" = "parquet",
    "use_path_style"="true");

DESC v1;

SELECT * FROM v1;

GRANT SELECT_PRIV ON db1.v1 TO user1;
```

### Data Ingestion

Users can ingest files into Doris tables via  `INSERT INTO SELECT`  for faster file analysis:

```sql
// 1. Create Doris internal table
CREATE TABLE IF NOT EXISTS test_table
(
    id int,
    name varchar(50),
    age int
)
DISTRIBUTED BY HASH(id) BUCKETS 4
PROPERTIES("replication_num" = "1");

// 2. Insert data using S3 Table Value Function
INSERT INTO test_table (id,name,age)
SELECT cast(id as INT) as id, name, cast (age as INT) as age
FROM s3(
    "uri" = "http://127.0.0.1:9312/test2/test.snappy.parquet",
    "s3.access_key"= "ak",
    "s3.secret_key" = "sk",
    "format" = "parquet",
    "use_path_style" = "true");
```


### Note

1. If the URI specified by the `S3 / HDFS` TVF is not matched with the file, or all the matched files are empty files, then the` S3 / HDFS` TVF will return to the empty result set. In this case, using the `DESC FUNCTION` to view the schema of this file, you will get a dummy column` __dummy_col`, which can be ignored.

2. If the format of the TVF is specified to `CSV`, and the read file is not a empty file but the first line of this file is empty, then it will prompt the error `The first line is empty, can not parse column numbers`. This is because the schema cannot be parsed from the first line of the file
