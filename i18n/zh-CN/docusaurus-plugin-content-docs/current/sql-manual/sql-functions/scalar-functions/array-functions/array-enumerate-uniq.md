---
{
    "title": "ARRAY_ENUMERATE_UNIQ",
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

## 描述
返回与源数组大小相同的数组，指示每个元素在具有相同值的元素中的位置，例如 array_enumerate_uniq([1, 2, 1, 4]) = [1, 1, 2, 1]
该函数也可接受多个大小相同的数组作为参数，这种情况下，返回的是数组中相同位置的元素组成的元组在具有相同值的元组中的位置。例如 array_enumerate_uniq([1, 2, 1, 1, 2], [2, 1, 2, 2, 1]) = [1, 1, 2, 3, 2]

## 语法
```sql
ARRAY_ENUMERATE_UNIQ(<arr1> [,<arr2> , ... ])
```
## 参数
| Parameter | Description |
|---|---|
| `<arr1>` | 需要计算的数组 arr1 |
| `<arr2>` | 需要计算的数组 arr2 |

## 返回值
返回与源数组大小相同的数组，指示每个元素在具有相同值的元素中的位置。

## 举例

```sql
select array_enumerate_uniq([1, 2, 3, 1, 2, 3]);
```
```text
+-----------------------------------------------------+
| array_enumerate_uniq(ARRAY(1, 2, 3, 1, 2, 3))       |
+-----------------------------------------------------+
| [1, 1, 1, 2, 2, 2]                                  |
+-----------------------------------------------------+
```
```sql
select array_enumerate_uniq([1, 1, 1, 1, 1], [2, 1, 2, 1, 2], [3, 1, 3, 1, 3]);
```
```text
+----------------------------------------------------------------------------------------+
| array_enumerate_uniq(ARRAY(1, 1, 1, 1, 1), ARRAY(2, 1, 2, 1, 2), ARRAY(3, 1, 3, 1, 3)) |
+----------------------------------------------------------------------------------------+
| [1, 1, 2, 1, 3]                                                                        |
+----------------------------------------------------------------------------------------+
```
