---
{
    "title": "FROM_UNIXTIME",
    "language": "en"
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

## from_unixtime
### description
#### syntax

`DATETIME FROM UNIXTIME (INT unix timestamp [, VARCHAR string format]`

Convert the UNIX timestamp to the corresponding time format of bits, and the format returned is specified by `string_format`

Input is an integer and return is a string type

Support `date_format`'s format, and default is `%Y-%m-%d %H:%i:%s`

### example

```
mysql> select from_unixtime(1196440219);
+---------------------------+
| from_unixtime(1196440219) |
+---------------------------+
| 2007-12-01 00:30:19       |
+---------------------------+

mysql> select from_unixtime(1196440219, '%Y-%m-%d');
+-----------------------------------------+
| from_unixtime(1196440219, '%Y-%m-%d')   |
+-----------------------------------------+
| 2007-12-01                              |
+-----------------------------------------+

mysql> select from_unixtime(1196440219, '%Y-%m-%d %H:%i:%s');
+--------------------------------------------------+
|From unixtime (1196440219,'%Y-%m-%d %H:%i:%s')    |
+--------------------------------------------------+
| 2007-12-01 00:30:19                              |
+--------------------------------------------------+
```

### keywords

    FROM_UNIXTIME,FROM,UNIXTIME
