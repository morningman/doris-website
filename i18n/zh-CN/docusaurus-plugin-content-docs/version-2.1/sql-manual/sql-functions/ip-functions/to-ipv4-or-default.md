---
{
"title": "TO_IPV4_OR_DEFAULT",
"language": "zh-CN"
}
---

<!-- 
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at
  http://www.apache.org/licenses/LICENSE-2.0
Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
-->

## TO_IPV4_OR_DEFAULT

TO_IPV4_OR_DEFAULT

## 描述

## 语法

`IPV4 TO_IPV4_OR_DEFAULT(STRING ipv4_str)`

与to_ipv4函数类似，但如果IPv4地址的格式非法，则返回0.0.0.0。

### 注意事项

入参如果为 `NULL`，则返回 `0.0.0.0。`

## 举例

```sql
mysql> select to_ipv4_or_default('.');
+-------------------------+
| to_ipv4_or_default('.') |
+-------------------------+
| 0.0.0.0                 |
+-------------------------+

mysql> select to_ipv4_or_default(NULL);
+--------------------------+
| to_ipv4_or_default(NULL) |
+--------------------------+
| 0.0.0.0                  |
+--------------------------+
```

### Keywords

TO_IPV4_OR_DEFAULT, IP
