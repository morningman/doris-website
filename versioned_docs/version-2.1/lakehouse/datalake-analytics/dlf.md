---
{
    "title": "Alibaba Cloud DLF",
    "language": "en"
}
---

# Alibaba Cloud DLF

Data Lake Formation (DLF) is the unified metadata management service of Alibaba Cloud. It is compatible with the Hive Metastore protocol.

> [What is DLF](https://www.alibabacloud.com/product/datalake-formation)

Doris can access DLF the same way as it accesses Hive Metastore.

## Connect to DLF

### Create a DLF Catalog.

```sql
CREATE CATALOG dlf PROPERTIES (
   "type"="hms",
   "hive.metastore.type" = "dlf",
   "dlf.proxy.mode" = "DLF_ONLY",
   "dlf.endpoint" = "datalake-vpc.cn-beijing.aliyuncs.com",
   "dlf.region" = "cn-beijing",
   "dlf.uid" = "uid",
   "dlf.catalog.id" = "catalog_id", //optional
   "dlf.access_key" = "ak",
   "dlf.secret_key" = "sk"
);
```

`type` should always be `hms`. If you need to access Alibaba Cloud OSS on the public network, can add `"dlf.access.public"="true"`.

* `dlf.endpoint`: DLF Endpoint. See [Regions and Endpoints of DLF](https://www.alibabacloud.com/help/en/data-lake-formation/latest/regions-and-endpoints).
* `dlf.region`: DLF Region. See [Regions and Endpoints of DLF](https://www.alibabacloud.com/help/en/data-lake-formation/latest/regions-and-endpoints).
* `dlf.uid`: Alibaba Cloud account. You can find the "Account ID" in the upper right corner on the Alibaba Cloud console.
* `dlf.catalog.id`: Optional. Used to specify the dlf catalog, if not specified, the default Catalog ID will be used.
* `dlf.access_key`：AccessKey, which you can create and manage on the [Alibaba Cloud console](https://ram.console.aliyun.com/manage/ak).
* `dlf.secret_key`：SecretKey, which you can create and manage on the [Alibaba Cloud console](https://ram.console.aliyun.com/manage/ak).

Other configuration items are fixed and require no modifications.

After the above steps, you can access metadata in DLF the same way as you access Hive MetaStore.

Doris supports accessing Hive/Iceberg/Hudi metadata in DLF.

### Use OSS-HDFS as the datasource

1. Enable OSS-HDFS. [Grant access to OSS or OSS-HDFS](https://www.alibabacloud.com/help/en/e-mapreduce/latest/oss-hdfsnew)
2. Download the SDK. [JindoData SDK](https://github.com/aliyun/alibabacloud-jindodata/blob/master/docs/user/6.x/6.7.8/jindodata_download.md). If the Jindo SDK directory already exists on the cluster, skip this step.
3. Decompress the jindosdk.tar.gz or locate the Jindo SDK directory on the cluster, and then enter its lib directory and put `jindo-core.jar, jindo-sdk.jar` to both `${DORIS_HOME}/fe/lib` and `${DORIS_HOME}/be/lib/java_extensions/preload-extensions`.
4. Create DLF Catalog, set `oss.hdfs.enabled` as `true`：

    ```sql
    CREATE CATALOG dlf_oss_hdfs PROPERTIES (
       "type"="hms",
       "hive.metastore.type" = "dlf",
       "dlf.proxy.mode" = "DLF_ONLY",
       "dlf.endpoint" = "datalake-vpc.cn-beijing.aliyuncs.com",
       "dlf.region" = "cn-beijing",
       "dlf.uid" = "uid",
       "dlf.catalog.id" = "catalog_id", //optional
       "dlf.access_key" = "ak",
       "dlf.secret_key" = "sk",
       "oss.hdfs.enabled" = "true"
    );
    ```

5. When the Jindo SDK version is inconsistent with the version used on the EMR cluster, will reported `Plugin not found` and the Jindo SDK  needs to be replaced with the corresponding version.

### DLF Iceberg Catalog

```sql
CREATE CATALOG dlf_iceberg PROPERTIES (
   "type"="iceberg",
   "iceberg.catalog.type" = "dlf",
   "dlf.proxy.mode" = "DLF_ONLY",
   "dlf.endpoint" = "datalake-vpc.cn-beijing.aliyuncs.com",
   "dlf.region" = "cn-beijing",
   "dlf.uid" = "uid",
   "dlf.catalog.id" = "catalog_id", //optional
   "dlf.access_key" = "ak",
   "dlf.secret_key" = "sk"
);
```

## Column type mapping

Consistent with Hive Catalog, please refer to the **column type mapping** section in [Hive Catalog](./hive.md).

