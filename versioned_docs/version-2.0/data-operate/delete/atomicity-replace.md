---
{
    "title": "Replacing Atomic Table",
    "language": "en"
}
---

# Atomicity Replace

Doris supports atomic table replacement operations for two tables. This is only applicable to OLAP tables.

## Applicable scenarios

- Atomic overwrite operations
- In certain cases, users may want to rewrite data in a table. However, the "delete and load" approach causes a data invisibility window. To solve that, Doris allows users to create a new table of the same schema using the CREATE TABLE LIKE statement, import the new data into this new table, and then atomically replace the old table with the new table. For atomic replacement at the partition level, please refer to the [temporary partition](../../data-operate/delete/table-temp-partition/)documentation.

## Syntax

```Plain
ALTER TABLE [db.]tbl1 REPLACE WITH TABLE tbl2
[PROPERTIES('swap' = 'true')];
```

Replace table tbl1 with table tbl2.

If `swap` is `true`, after the replacement, data in `tbl1` will be replaced by that in `tbl2`, while data in `tbl2` will be replaced by that in `tbl1`. In other words, the two tables will swap data.

If `swap` is `false`, after the replacement, data in `tbl1` will be replaced by that in `tbl2` and `tbl2` will be deleted.

## Implementation

In fact, table replacement is to combine the following operations into one atomic operation.

Assuming that table A is to be replaced with table B, and `swap` is set to `true`. The operations to be implemented are as follows:

1. Rename table B to table A.
2. Rename table A to table B.

If `swap` is set to `false`, the operations are as follows:

1. Delete table A.
2. Rename table B to table A.

## Note

- `swap` defaults to `true`, meaning to swap the data between two tables.
- If `swap` is set to `false`, the table being replaced (table A) will be deleted and cannot be recovered.
- The replacement operation can only be implemented between two OLAP tables and it does not check for table schema consistency.
- The replacement operation does not change the existing privilege settings because privilege checks are based on table names.
