---
{
    "title": "Resource management",
    "language": "en"
}
---

# Resource Management

In order to save the compute and storage resources in the Doris cluster, Doris needs to reference to some other external resources to do the related work. such as spark/GPU for query, HDFS/S3 for external storage, spark/MapReduce for ETL, connect to external storage by ODBC driver. Therefore, Doris need a resource management mechanism to manage these external resources.

## Fundamental Concept

A resource contains basic information such as name and type. The name is globally unique. Different types of resources contain different attributes. Please refer to the introduction of each resource for details.

The creation and deletion of resources can only be performed by users own `admin` permission. One resource belongs to the entire Doris cluster. Users with `admin` permission can assign permission of resource to other users. Please refer to `HELP GRANT` or doris document.


## Operation Of Resource

There are three main commands for resource management: `create resource`, `drop resource` and `show resources`. They are to create, delete and check resources. The specific syntax of these three commands can be viewed by executing `help CMD` after MySQL client connects to Doris.

1. CREATE RESOURCE

    This statement is used to create a resource. For details, please refer to [CREATE RESOURCE](../sql-manual/sql-reference/Data-Definition-Statements/Create/CREATE-RESOURCE.md).

2. DROP RESOURCE

    This command can delete an existing resource. For details, see [DROP RESOURCE](../sql-manual/sql-reference/Data-Definition-Statements/Drop/DROP-RESOURCE.md).

3. SHOW RESOURCES

    This command can view the resources that the user has permission to use. For details, see [SHOW RESOURCES](../sql-manual/sql-reference/Show-Statements/SHOW-RESOURCES.md).

## Resources Supported

Currently, Doris can support

* Spark resource: do ETL work
* ODBC resource: query and import data from external tables

The following shows how the two resources are used.

### Spark

#### Parameter

##### Spark Parameters:

`spark.master`: required, currently supported yarn, spark://host:port.

`spark.submit.deployMode`: The deployment mode of spark. required. It supports cluster and client.

`spark.hadoop.yarn.resourcemanager.address`: required when master is yarn.

`spark.hadoop.fs.defaultFS`: required when master is yarn.

Other parameters are optional, refer to: http://spark.apache.org/docs/latest/configuration.html

##### If spark is used for ETL, also need to specify the following parameters:

`working_dir`: Directory used by ETL. Spark is required when used as an ETL resource. For example: hdfs://host:port/tmp/doris.

`broker`: The name of broker. Is required when spark be used as ETL resource. You need to use the `ALTER SYSTEM ADD BROKER` command to complete the configuration in advance. 

  * `broker.property_key`: When the broker reads the intermediate file generated by ETL, it needs the specified authentication information.



#### Example

Create a spark resource named `spark0 `in the yarn cluster mode.


```sql
CREATE EXTERNAL RESOURCE "spark0"
PROPERTIES
(
  "type" = "spark",
  "spark.master" = "yarn",
  "spark.submit.deployMode" = "cluster",
  "spark.jars" = "xxx.jar,yyy.jar",
  "spark.files" = "/tmp/aaa,/tmp/bbb",
  "spark.executor.memory" = "1g",
  "spark.yarn.queue" = "queue0",
  "spark.hadoop.yarn.resourcemanager.address" = "127.0.0.1:9999",
  "spark.hadoop.fs.defaultFS" = "hdfs://127.0.0.1:10000",
  "working_dir" = "hdfs://127.0.0.1:10000/tmp/doris",
  "broker" = "broker0",
  "broker.username" = "user0",
  "broker.password" = "password0"
);
```

### ODBC

#### Parameter

##### ODBC Parameters:

`type`: Required, must be `odbc_catalog`. As the type identifier of resource.

`user`: The user name of the external table, required.

`password`: The user password of the external table, required.

`host`: The ip address of the external table, required.

`port`: The port of the external table, required.

`odbc_type`: Indicates the type of external table. Currently, Doris supports `MySQL` and `Oracle`. In the future, it may support more databases. The ODBC external table referring to the resource is required. The old MySQL external table referring to the resource is optional.

`driver`: Indicates the driver dynamic library used by the ODBC external table.
The ODBC external table referring to the resource is required. The old MySQL external table referring to the resource is optional.

For the usage of ODBC resource, please refer to [ODBC of Doris](../lakehouse/external-table/odbc.md)


#### Example

Create the ODBC resource of Oracle, named `oracle_odbc`.

```sql
CREATE EXTERNAL RESOURCE `oracle_odbc`
PROPERTIES (
"type" = "odbc_catalog",
"host" = "192.168.0.1",
"port" = "8086",
"user" = "test",
"password" = "test",
"database" = "test",
"odbc_type" = "oracle",
"driver" = "Oracle 19 ODBC driver"
);
```
