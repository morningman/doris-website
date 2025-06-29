---
{
  "title": "Huawei OBS",
  "language": "en"
}
---

This document introduces the parameters required to access Huawei Cloud OBS, applicable to the following scenarios:

- Catalog properties
- Table Valued Function properties
- Broker Load properties
- Export properties
- Outfile properties

**Doris uses the S3 Client to access Huawei Cloud OBS through the S3 compatible protocol.**

## Parameter Overview

| Property Name                     | Former Name      | Description                                | Default | Required |
|-----------------------------------|------------------|--------------------------------------------|---------|----------|
| `s3.endpoint`                     | `obs.endpoint`   | OBS endpoint, specifies the access endpoint of Huawei Cloud OBS |         | Yes      |
| `s3.region`                       | `obs.region`     | OBS region, specifies the region of Huawei Cloud OBS |         | No       |
| `s3.access_key`                   | `obs.access_key` | OBS access key, the access key for authentication |         | Yes      |
| `s3.secret_key`                   | `obs.secret_key` | OBS secret key, the secret key used with the access key |         | Yes      |
| `s3.connection.maximum`           |                  | Maximum number of S3 connections, specifies the maximum number of connections established with the OBS service | `50`    | No       |
| `s3.connection.request.timeout`   |                  | S3 request timeout in milliseconds, specifies the request timeout when connecting to the OBS service | `3000`  | No       |
| `s3.connection.timeout`           |                  | S3 connection timeout in milliseconds, specifies the timeout when establishing a connection with the OBS service | `1000`  | No       |

### Authentication Configuration

When accessing Huawei Cloud OBS, you need to provide Huawei Cloud's Access Key and Secret Key, which are the following parameters:

- `s3.access_key` (or `obs.access_key`)
- `s3.secret_key` (or `obs.secret_key`)

These two parameters are used for authentication to ensure access permissions to Huawei Cloud OBS.

### Configuration Example

```plaintext:
"s3.access_key" = "ak",
"s3.secret_key" = "sk",
"s3.endpoint" = "obs.cn-north-4.myhuaweicloud.com"
"s3.region" = "cn-north-4"
```

