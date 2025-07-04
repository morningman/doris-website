---
{
    "title": "TIME_ROUND",
    "language": "en"
}
---

## time_round
### description
#### Syntax

```sql
DATETIME TIME_ROUND(DATETIME expr)
DATETIME TIME_ROUND(DATETIME expr, INT period)
DATETIME TIME_ROUND(DATETIME expr, DATETIME origin)
DATETIME TIME_ROUND(DATETIME expr, INT period, DATETIME origin)
```

The function name `TIME_ROUND` consists of two parts, Each part consists of the following optional values.
- `TIME`: `SECOND`, `MINUTE`, `HOUR`, `DAY`, `WEEK`, `MONTH`, `YEAR`
- `ROUND`: `FLOOR`, `CEIL`

Returns the upper/lower bound of `expr`.

- `period` specifies how many `TIME` units, the default is `1`.
- `origin` specifies the start time of the period, the default is `1970-01-01T00:00:00`, the start time of `WEEK` is Sunday, which is `1970-01-04T00:00:00`. Could be larger than `expr`.
- Please try to choose common `period`, such as 3 `MONTH`, 90 `MINUTE`. If you set a uncommon `period`, please also specify `origin`.

### example

```

MySQL> SELECT YEAR_FLOOR('20200202000000');
+------------------------------+
| year_floor('20200202000000') |
+------------------------------+
| 2020-01-01 00:00:00          |
+------------------------------+


MySQL> SELECT MONTH_CEIL(CAST('2020-02-02 13:09:20' AS DATETIME), 3); --quarter
+--------------------------------------------------------+
| month_ceil(CAST('2020-02-02 13:09:20' AS DATETIME), 3) |
+--------------------------------------------------------+
| 2020-04-01 00:00:00                                    |
+--------------------------------------------------------+


MySQL> SELECT WEEK_CEIL('2020-02-02 13:09:20', '2020-01-06'); --monday
+---------------------------------------------------------+
| week_ceil('2020-02-02 13:09:20', '2020-01-06 00:00:00') |
+---------------------------------------------------------+
| 2020-02-03 00:00:00                                     |
+---------------------------------------------------------+


MySQL> SELECT MONTH_CEIL(CAST('2020-02-02 13:09:20' AS DATETIME), 3, CAST('1970-01-09 00:00:00' AS DATETIME)); --next rent day
+-------------------------------------------------------------------------------------------------+
| month_ceil(CAST('2020-02-02 13:09:20' AS DATETIME), 3, CAST('1970-01-09 00:00:00' AS DATETIME)) |
+-------------------------------------------------------------------------------------------------+
| 2020-04-09 00:00:00                                                                             |
+-------------------------------------------------------------------------------------------------+

```
### keywords
    TIME_ROUND
