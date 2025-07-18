---
{
  "title": "Tencent COS",
  "language": "en"
}
---

This document introduces the parameters required to access Tencent Cloud COS, applicable to the following scenarios:

- Catalog properties
- Table Valued Function properties
- Broker Load properties
- Export properties
- Outfile properties

**Doris uses the S3 Client to access Tencent Cloud COS through the S3 compatible protocol.**

## Parameter Overview

| Property Name                     | Former Name      | Description                                | Default Value | Required |
|-----------------------------------|------------------|--------------------------------------------|---------------|----------|
| `s3.endpoint`                     | `cos.endpoint`   | COS endpoint, specifies the access endpoint of Tencent Cloud COS |               | Yes      |
| `s3.region`                       | `cos.region`     | COS region, specifies the region of Tencent Cloud COS |               | No       |
| `s3.access_key`                   | `cos.access_key` | COS access key, the access key for authentication |               | Yes      |
| `s3.secret_key`                   | `cos.secret_key` | COS secret key, the secret key used with the access key |               | Yes      |
| `s3.connection.maximum`           |                  | Maximum S3 connections, specifies the maximum number of connections to the COS service | `50`          | No       |
| `s3.connection.request.timeout`   |                  | S3 request timeout, in milliseconds, specifies the request timeout when connecting to the COS service | `3000`        | No       |
| `s3.connection.timeout`           |                  | S3 connection timeout, in milliseconds, specifies the timeout when establishing a connection to the COS service | `1000`        | No       |
| `s3.sts_endpoint`                 |                  | Not supported yet                           |               | No       |
| `s3.sts_region`                   |                  | Not supported yet                           |               | No       |
| `s3.iam_role`                     |                  | Not supported yet                           |               | No       |
| `s3.external_id`                  |                  | Not supported yet                           |               | No       |

### Authentication Configuration

When accessing Tencent Cloud COS, you need to provide Tencent Cloud's Access Key and Secret Key, which are the following parameters:

- `s3.access_key` (or `cos.access_key`)
- `s3.secret_key` (or `cos.secret_key`)

### Example Configuration

```plaintext
"cos.access_key" = "ak",
"cos.secret_key" = "sk",
"cos.endpoint" = "cos.ap-beijing.myqcloud.com",
"cos.region" = "ap-beijing"
```

