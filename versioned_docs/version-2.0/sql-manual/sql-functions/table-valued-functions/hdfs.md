---
{
    "title": "HDFS",
    "language": "en"
}
---

## HDFS

### Name

hdfs

### Description

HDFS table-valued-function(tvf), allows users to read and access file contents on S3-compatible object storage, just like accessing relational table. Currently supports `csv/csv_with_names/csv_with_names_and_types/json/parquet/orc` file format.

#### syntax

```sql
hdfs(
  "uri" = "..",
  "fs.defaultFS" = "...",
  "hadoop.username" = "...",
  "format" = "csv",
  "keyn" = "valuen" 
  ...
  );
```

**parameter description**

Related parameters for accessing hdfs:

- `uri`: (required) hdfs uri. If the uri path does not exist or the files are empty files, hdfs tvf will return an empty result set.
- `fs.defaultFS`: (required)
- `hadoop.username`: (required) Can be any string, but cannot be empty.
- `hadoop.security.authentication`: (optional)
- `hadoop.username`: (optional)
- `hadoop.kerberos.principal`: (optional)
- `hadoop.kerberos.keytab`: (optional)
- `dfs.client.read.shortcircuit`: (optional)
- `dfs.domain.socket.path`: (optional)

Related parameters for accessing HDFS in HA mode:

- `dfs.nameservices`: (optional)
- `dfs.ha.namenodes.your-nameservices`: (optional)
- `dfs.namenode.rpc-address.your-nameservices.your-namenode`: (optional)
- `dfs.client.failover.proxy.provider.your-nameservices`: (optional)

File format parameters:

- `format`: (required) Currently support `csv/csv_with_names/csv_with_names_and_types/json/parquet/orc/avro`
- `column_separator`: (optional) default `\t`.
- `line_delimiter`: (optional) default `\n`.
- `compress_type`: (optional) Currently support `UNKNOWN/PLAIN/GZ/LZO/BZ2/LZ4FRAME/DEFLATE/SNAPPYBLOCK`. Default value is `UNKNOWN`, it will automatically infer the type based on the suffix of `uri`.

    The following 6 parameters are used for loading in json format. For specific usage methods, please refer to: [Json Load](../../../data-operate/import/file-format/json)

- `read_json_by_line`: (optional) default `"true"`
- `strip_outer_array`: (optional) default `"false"`
- `json_root`: (optional) default `""`
- `json_paths`: (optional) default `""`
- `num_as_string`: (optional) default `false`
- `fuzzy_parse`: (optional) default `false`

  The following 2 parameters are used for loading in csv format:

- `trim_double_quotes`: Boolean type (optional), the default value is `false`. True means that the outermost double quotes of each field in the csv file are trimmed.
- `skip_lines`: Integer type (optional), the default value is 0. It will skip some lines in the head of csv file. It will be disabled when the format is `csv_with_names` or `csv_with_names_and_types`.

other kinds of parameters:

- `path_partition_keys`: (optional) Specifies the column names carried in the file path. For example, if the file path is /path/to/city=beijing/date="2023-07-09", you should fill in `path_partition_keys="city,date"`. It will automatically read the corresponding column names and values from the path during load process.
- `resource`：（optional）Specify the resource name. Hdfs Tvf can use the existing Hdfs resource to directly access Hdfs. You can refer to the method for creating an Hdfs resource: [CREATE-RESOURCE](../../sql-statements/Data-Definition-Statements/Create/CREATE-RESOURCE.md). This property is supported starting from version 2.1.4. 

:::tip Tip
To directly query a TVF or create a VIEW based on that TVF, you need to have usage permission for that resource. To query a VIEW created based on TVF, you only need select permission for that VIEW.
:::

### Examples

Read and access csv format files on hdfs storage.

```sql
MySQL [(none)]> select * from hdfs(
            "uri" = "hdfs://127.0.0.1:842/user/doris/csv_format_test/student.csv",
            "fs.defaultFS" = "hdfs://127.0.0.1:8424",
            "hadoop.username" = "doris",
            "format" = "csv");
+------+---------+------+
| c1   | c2      | c3   |
+------+---------+------+
| 1    | alice   | 18   |
| 2    | bob     | 20   |
| 3    | jack    | 24   |
| 4    | jackson | 19   |
| 5    | liming  | 18   |
+------+---------+------+
```

Read and access csv format files on hdfs storage in HA mode.

```sql
MySQL [(none)]> select * from hdfs(
            "uri" = "hdfs://127.0.0.1:842/user/doris/csv_format_test/student.csv",
            "fs.defaultFS" = "hdfs://127.0.0.1:8424",
            "hadoop.username" = "doris",
            "format" = "csv",
            "dfs.nameservices" = "my_hdfs",
            "dfs.ha.namenodes.my_hdfs" = "nn1,nn2",
            "dfs.namenode.rpc-address.my_hdfs.nn1" = "nanmenode01:8020",
            "dfs.namenode.rpc-address.my_hdfs.nn2" = "nanmenode02:8020",
            "dfs.client.failover.proxy.provider.my_hdfs" = "org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider");
+------+---------+------+
| c1   | c2      | c3   |
+------+---------+------+
| 1    | alice   | 18   |
| 2    | bob     | 20   |
| 3    | jack    | 24   |
| 4    | jackson | 19   |
| 5    | liming  | 18   |
+------+---------+------+
```

Can be used with `desc function` :

```sql
MySQL [(none)]> desc function hdfs(
            "uri" = "hdfs://127.0.0.1:8424/user/doris/csv_format_test/student_with_names.csv",
            "fs.defaultFS" = "hdfs://127.0.0.1:8424",
            "hadoop.username" = "doris",
            "format" = "csv_with_names");
```

### Keywords

    hdfs, table-valued-function, tvf

### Best Practice

  For more detailed usage of HDFS tvf, please refer to [S3](./s3.md) tvf, The only difference between them is the way of accessing the storage system.
