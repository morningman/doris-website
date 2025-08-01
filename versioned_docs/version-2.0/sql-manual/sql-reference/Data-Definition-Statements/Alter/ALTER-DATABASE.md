---
{
    "title": "ALTER-DATABASE",
    "language": "en"
}
---

## ALTER-DATABASE

### Name

ALTER DATABASE

### Description

This statement is used to set properties of the specified database. (administrator only)

1) Set the database data quota, the unit is B/K/KB/M/MB/G/GB/T/TB/P/PB

```sql
ALTER DATABASE db_name SET DATA QUOTA quota;
```

2) Rename the database

```sql
ALTER DATABASE db_name RENAME new_db_name;
```

3) Set the quota for the number of copies of the database

```sql
ALTER DATABASE db_name SET REPLICA QUOTA quota;
```

illustrate:
    After renaming the database, use the REVOKE and GRANT commands to modify the appropriate user permissions, if necessary.
    The default data quota for the database is 1PB, and the default replica quota is 1073741824.

4) Modify the properties of an existing database

```sql
ALTER DATABASE db_name SET PROPERTIES ("key"="value", ...); 
```

### Example

1. Set the specified database data volume quota

```sql
ALTER DATABASE example_db SET DATA QUOTA 10995116277760;
The above unit is bytes, which is equivalent to
ALTER DATABASE example_db SET DATA QUOTA 10T;

ALTER DATABASE example_db SET DATA QUOTA 100G;

ALTER DATABASE example_db SET DATA QUOTA 200M;
```

2. Rename the database example_db to example_db2

```sql
ALTER DATABASE example_db RENAME example_db2;
```

3. Set the quota for the number of copies of the specified database

```sql
ALTER DATABASE example_db SET REPLICA QUOTA 102400;
```

4. Modify the default replica distribution policy for tables in db (this operation only applies to newly created tables and will not modify existing tables in db)

```sql
ALTER DATABASE example_db SET PROPERTIES("replication_allocation" = "tag.location.default:2");
```

5. Cancel the default replica distribution policy for tables in db (this operation only applies to newly created tables and will not modify existing tables in db)

```sql
ALTER DATABASE example_db SET PROPERTIES("replication_allocation" = "");
```

### Keywords

```text
ALTER,DATABASE,RENAME
```

### Best Practice

