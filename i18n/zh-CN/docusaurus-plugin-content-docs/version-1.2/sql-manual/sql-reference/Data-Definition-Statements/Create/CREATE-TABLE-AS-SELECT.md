---
{
    "title": "CREATE-TABLE-AS-SELECT",
    "language": "zh-CN"
}
---

## CREATE-TABLE-AS-SELECT

### Name

CREATE TABLE AS SELECT

## 描述

该语句通过 Select 语句返回结果创建表结构，同时导入数据

语法：

```sql
CREATE TABLE table_name [( column_name_list )]
    opt_engine:engineName
    opt_keys:keys
    opt_comment:tableComment
    opt_partition:partition
    opt_distribution:distribution
    opt_rollup:index
    opt_properties:tblProperties
    opt_ext_properties:extProperties
    KW_AS query_stmt:query_def
 ```

说明: 

- 用户需要拥有来源表的`SELECT`权限和目标库的`CREATE`权限
- 创建表成功后，会进行数据导入，如果导入失败，将会删除表
- 可以自行指定 key type，默认为`Duplicate Key`

<version since='1.2'>

- 所有字符串类型的列(varchar/var/string) 都会被创建为 string 类型。
- 如果创建的来源为外部表，并且第一列为 String 类型，则会自动将第一列设置为 VARCHAR(65533)。因为 Doris 内部表，不允许 String 列作为第一列。

</version>

## 举例

1. 使用 select 语句中的字段名

    ```sql
    create table `test`.`select_varchar` 
    PROPERTIES(\"replication_num\" = \"1\") 
    as select * from `test`.`varchar_table`
    ```

2. 自定义字段名(需要与返回结果字段数量一致)
    ```sql
    create table `test`.`select_name`(user, testname, userstatus) 
    PROPERTIES(\"replication_num\" = \"1\") 
    as select vt.userId, vt.username, jt.status 
    from `test`.`varchar_table` vt join 
    `test`.`join_table` jt on vt.userId=jt.userId
    ```
   
### Keywords

    CREATE, TABLE, AS, SELECT

### Best Practice

