---
{
    "title": "PERCENTILE_ARRAY",
    "language": "en"
}
---

## PERCENTILE_ARRAY
### Description
#### Syntax

`ARRAY_DOUBLE PERCENTILE_ARRAY(BIGINT, ARRAY_DOUBLE p)`

Calculate exact percentiles, suitable for small data volumes. Sorts the specified column in descending order first, then takes the exact pth percentile.
The return value is the result of sequentially taking the specified percentages in the array p.
Parameter Description:
expr: Required. Columns whose values are of type integer (up to bigint).
p: The exact percentile is required, an array of constants, taking the value [0.0, 1.0].

### example
```
mysql> select percentile_array(k1,[0.3,0.5,0.9]) from baseall;
+----------------------------------------------+
| percentile_array(`k1`, ARRAY(0.3, 0.5, 0.9)) |
+----------------------------------------------+
| [5.2, 8, 13.6]                               |
+----------------------------------------------+
1 row in set (0.02 sec)

```

### keywords
PERCENTILE_ARRAY
