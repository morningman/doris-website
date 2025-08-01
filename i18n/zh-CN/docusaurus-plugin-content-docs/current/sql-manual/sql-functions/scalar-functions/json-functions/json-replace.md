---
{
    "title": "JSON_REPLACE",
    "language": "zh-CN"
}
---

## 描述
`JSON_REPLACE` 函数用于在 JSON 中插入数据并返回结果。

## 语法
```sql
JSON_REPLACE (<json_object>, <path>,  <value>[, <path>,  <value>, ...])
```

## 参数
- `<json_object>` JSON 类型表达式，被修改的目标。
- `<path>` String 类型表达式，指定替换值的路径
- `<value>` JSON 类型或其他 [`TO_JSON`](./to-json.md) 支持的类型，要替换的值。

## 返回值
- `Nullable(JSON)` 返回被修改后的 JSON 对象

## 使用说明
1. 需要注意的是，路径值对按从左到右的顺序进行评估。
2. 如果 `<path>` 指向的值在 JSON 对象中不存在，不会产生任何影响。
3. `<path>` 中不能包含通配符，如果包含通配符会报错。
4. `<json_object>` 或者 `<path>` 为 NULL 时，会得到 NULL，如果 `<value>` 为 NULL 会插入一个 JSON 的 null 值。

## 示例
1. 路径值对按从左到右的顺序进行评估
    ```sql
    select json_replace('{"k": {"k2": "v2"}}', '$.k', json_parse('{"k2": 321, "k3": 456}'), '$.k.k2', 123);
    ```
    ```text
    +-------------------------------------------------------------------------------------------------+
    | json_replace('{"k": {"k2": "v2"}}', '$.k', json_parse('{"k2": 321, "k3": 456}'), '$.k.k2', 123) |
    +-------------------------------------------------------------------------------------------------+
    | {"k":{"k2":123,"k3":456}}                                                                       |
    +-------------------------------------------------------------------------------------------------+
    ```
2. `<path>` 指向的值在 JSON 对象中不存在
    ```sql
    select json_replace('{"k": 1}', "$.k2", 2);
    ```
    ```text
    +-------------------------------------+
    | json_replace('{"k": 1}', "$.k2", 2) |
    +-------------------------------------+
    | {"k":1}                             |
    +-------------------------------------+
    ```
3. `<path>` 不能包含通配符
    ```sql
    select json_replace('{"k": 1}', "$.*", 2);
    ```
    ```text
    ERROR 1105 (HY000): errCode = 2, detailMessage = [INVALID_ARGUMENT] In this situation, path expressions may not contain the * and ** tokens or an array range, argument index: 1, row index: 0
    ```
4. NULL 参数
    ```sql
    select json_replace(NULL, '$[1]', 123);
    ```
    ```text
    +---------------------------------+
    | json_replace(NULL, '$[1]', 123) |
    +---------------------------------+
    | NULL                            |
    +---------------------------------+
    ```
    ```sql
    select json_replace('{"k": "v"}', NULL, 123);
    ```
    ```text
    +---------------------------------------+
    | json_replace('{"k": "v"}', NULL, 123) |
    +---------------------------------------+
    | NULL                                  |
    +---------------------------------------+
    ```
    ```sql
    select json_replace('{"k": "v"}', '$.k', NULL);
    ```
    ```text
    +-----------------------------------------+
    | json_replace('{"k": "v"}', '$.k', NULL) |
    +-----------------------------------------+
    | {"k":null}                              |
    +-----------------------------------------+
    ```