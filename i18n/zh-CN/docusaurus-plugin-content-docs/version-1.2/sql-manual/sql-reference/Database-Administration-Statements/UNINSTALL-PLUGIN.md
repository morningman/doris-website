---
{
    "title": "UNINSTALL-PLUGIN",
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

## UNINSTALL-PLUGIN

### Name

UNINSTALL PLUGIN

## 描述

该语句用于卸载一个插件。

语法：

```sql
UNINSTALL PLUGIN plugin_name;
```

 plugin_name 可以通过 `SHOW PLUGINS;` 命令查看。

只能卸载非 builtin 的插件。

## 举例

1. 卸载一个插件：

    ```sql
    UNINSTALL PLUGIN auditdemo;
    ```

### Keywords

    UNINSTALL, PLUGIN

### Best Practice

