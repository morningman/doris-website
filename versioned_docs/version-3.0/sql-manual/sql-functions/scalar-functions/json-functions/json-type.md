---
{
    "title": "JSON_TYPE",
    "language": "en"
}
---

## Description

Used to determine the type of the field specified by `json_path` in the JSONB data. If the field does not exist, it returns NULL. If the field exists, it returns one of the following types:

- object
- array
- null
- bool
- int
- bigint
- largeint
- double
- string

## Syntax

```sql
STRING JSON_TYPE( <JSON j> )
```

## Alias

- JSONB_TYPE

## Required Parameters


| Parameter | Description |
|------|------|
| `<JSON j>` | The JSON string to check the type of. |


## Return Value
Returns the type of the JSON string. Possible values include:
- "NULL": Indicates that the value in the JSON document is null.
- "BOOLEAN": Indicates that the value in the JSON document is of boolean type (true or false).
- "NUMBER": Indicates that the value in the JSON document is a number.
- "STRING": Indicates that the value in the JSON document is a string.
- "OBJECT": Indicates that the value in the JSON document is a JSON object.
- "ARRAY": Indicates that the value in the JSON document is a JSON array.

## Usage Notes

JSON_TYPE returns the type of the outermost value in the JSON document. If the JSON document contains multiple different types of values, it will return the type of the outermost value. For invalid JSON strings, JSON_TYPE returns NULL. Refer to [json tutorial](../../../basic-element/sql-data-types/semi-structured/JSON)

## Examples
1. JSON is of string type:

```sql
SELECT JSON_TYPE('"Hello, World!"');
```

```sql
+------------------------------------------+
| JSON_TYPE('"Hello, World!"')            |
+------------------------------------------+
| STRING                                   |
+------------------------------------------+
```

2. JSON is of number type:

```sql
SELECT JSON_TYPE('123');
```
```sql
+------------------------------------------+
| JSON_TYPE('123')                        |
+------------------------------------------+
| NUMBER                                   |
+------------------------------------------+
```
3. JSON is of null type:
```sql
SELECT JSON_TYPE('null');
```
```sql
+------------------------------------------+
| JSON_TYPE('null')                        |
+------------------------------------------+
| NULL                                     |
+------------------------------------------+
```
