---
{
    "title": "DATETIME",
    "language": "en"
}
---

## DATETIME

DATETIME

### Description

DATETIME([P])
Date and time type.
The optional parameter P indicates the time precision and the value range is [0, 6], that is, it supports up to 6 decimal places (microseconds). 0 when not set.
Value range is ['0000-01-01 00:00:00[.000000]','9999-12-31 23:59:59[.999999]'].
The form of printing is 'yyyy-MM-dd HH:mm:ss.SSSSSS'

### note

DATETIME supports temporal precision up to microseconds. When parsing imported DATETIME type data using the BE side (e.g. using Stream load, Spark load, etc.), or using the FE side with the [Nereids](/docs/query/nereids/nereids-new) on, decimals exceeding the current precision will be **rounded**.
Inserting a DATETIME value with a fractional seconds part into a column of the same type but having fewer fractional digits results in **rounded**.
DATETIME reads support resolving the time zone in the format of the original DATETIME literal followed by the time zone:
```sql
<date> <time>[<timezone>]
```

For the specific supported formats for `<timezone>`, see [timezone](../../../../admin-manual/cluster-management/time-zone). Note that the `DATE`, `DATEV2`, `DATETIME`, and `DATETIMEV2` types **don't** contain time zone information. For example, if an input time string "2012-12-12 08:00:00+08:00" is parsed and converted to the current time zone "+02:00", and the actual value "2012-12-12 02:00:00" is stored in the DATETIME column, the value itself will not change, no matter how much the cluster environment variables are changed.

### example

```sql
mysql> select @@time_zone;
+----------------+
| @@time_zone    |
+----------------+
| Asia/Hong_Kong |
+----------------+
1 row in set (0.11 sec)

mysql> insert into dtv23 values ("2020-12-12 12:12:12Z"), ("2020-12-12 12:12:12GMT"), ("2020-12-12 12:12:12+02:00"), ("2020-12-12 12:12:12America/Los_Angeles");
Query OK, 4 rows affected (0.17 sec)

mysql> select * from dtv23;
+-------------------------+
| k0                      |
+-------------------------+
| 2020-12-12 20:12:12.000 |
| 2020-12-12 20:12:12.000 |
| 2020-12-13 04:12:12.000 |
| 2020-12-12 18:12:12.000 |
+-------------------------+
4 rows in set (0.15 sec)
```
