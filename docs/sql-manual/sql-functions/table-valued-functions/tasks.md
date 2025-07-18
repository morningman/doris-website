---
{
    "title": "TASKS",
    "language": "en"
}
---

## Description

Table function, generates a temporary table of tasks, which allows you to view the information of tasks generated by jobs in the current Doris cluster.

## Syntax
```sql
TASKS(
    "type"="<type>"
)
```

## Required Parameters
| Field         | Description                                                                                       |
|---------------|---------------------------------------------------------------------------------------------------|
| **`<type>`**  | Type of the task: <br/> `insert`: insert into type task. <br/> `mv`: materialized view type task. |


## Return Value

-  **`tasks("type"="insert")`** tasks return value of type insert

   | Field Name   | Description                        |
   |--------------|------------------------------------|
   | **TaskId**   | Task id                            |
   | **JobId**    | Job id                             |
   | **JobName**  | Job name                           |
   | **Label**    | Label                              |
   | **Status**   | Task status                        |
   | **ErrorMsg** | Task failure information          |
   | **CreateTime**| Task creation time                |
   | **FinishTime**| Task completion time              |
   | **TrackingUrl**| Tracking URL                     |
   | **LoadStatistic**| Load statistics                |
   | **User**     | User                               |

-  **`tasks("type"="mv")`** Tasks return value of type MV

   | Field Name            | Description                                                                 |
   |-----------------------|-----------------------------------------------------------------------------|
   | **TaskId**            | Task id                                                                     |
   | **JobId**             | Job id                                                                      |
   | **JobName**           | Job Name                                                                    |
   | **MvId**              | Materialized View ID                                                        |
   | **MvName**            | Materialized View Name                                                      |
   | **MvDatabaseId**      | DB ID of the materialized view                                              |
   | **MvDatabaseName**    | Name of the database to which the materialized view belongs                 |
   | **Status**            | Task status                                                                 |
   | **ErrorMsg**          | Task failure information                                                   |
   | **CreateTime**        | Task creation time                                                          |
   | **StartTime**         | Task start running time                                                     |
   | **FinishTime**        | Task End Run Time                                                           |
   | **DurationMs**        | Task runtime                                                                |
   | **TaskContext**       | Task running parameters                                                     |
   | **RefreshMode**       | Refresh mode                                                                |
   | **NeedRefreshPartitions** | The partition information that needs to be refreshed for this task       |
   | **CompletedPartitions** | The partition information that has been refreshed for this task          |
   | **Progress**          | Task running progress                                                       |


## Examples

View tasks for all materialized views

```sql
select * from tasks("type"="mv");
```
```text
+-----------------+-------+------------------+-------+--------------------------+--------------+--------------------------------------------------------+---------+----------+---------------------+---------------------+---------------------+------------+-------------------------------------------------------------+-------------+-----------------------------------------------+-----------------------------------------------+---------------+-----------------------------------+
| TaskId          | JobId | JobName          | MvId  | MvName                   | MvDatabaseId | MvDatabaseName                                         | Status  | ErrorMsg | CreateTime          | StartTime           | FinishTime          | DurationMs | TaskContext                                                 | RefreshMode | NeedRefreshPartitions                         | CompletedPartitions                           | Progress      | LastQueryId                       |
+-----------------+-------+------------------+-------+--------------------------+--------------+--------------------------------------------------------+---------+----------+---------------------+---------------------+---------------------+------------+-------------------------------------------------------------+-------------+-----------------------------------------------+-----------------------------------------------+---------------+-----------------------------------+
| 509478985247053 | 23369 | inner_mtmv_23363 | 23363 | range_date_up_union_mv1  | 21805        | regression_test_nereids_rules_p0_mv_create_part_and_up | SUCCESS |          | 2025-01-08 18:19:10 | 2025-01-08 18:19:10 | 2025-01-08 18:19:10 | 233        | {"triggerMode":"SYSTEM","isComplete":false}                 | COMPLETE    | ["p_20231001_20231101"]                       | ["p_20231001_20231101"]                       | 100.00% (1/1) | 71897c47d0d94fd2-9ca52a0e6eb3bff5 |
| 509486915704885 | 23369 | inner_mtmv_23363 | 23363 | range_date_up_union_mv1  | 21805        | regression_test_nereids_rules_p0_mv_create_part_and_up | SUCCESS |          | 2025-01-08 18:19:17 | 2025-01-08 18:19:17 | 2025-01-08 18:19:17 | 227        | {"triggerMode":"MANUAL","partitions":[],"isComplete":false} | PARTIAL     | ["p_20231101_20231201"]                       | ["p_20231101_20231201"]                       | 100.00% (1/1) | 9bf5ff69d4cc4c78-b50505436c8410c4 |
| 509487197275880 | 23369 | inner_mtmv_23363 | 23363 | range_date_up_union_mv1  | 21805        | regression_test_nereids_rules_p0_mv_create_part_and_up | SUCCESS |          | 2025-01-08 18:19:18 | 2025-01-08 18:19:18 | 2025-01-08 18:19:18 | 191        | {"triggerMode":"MANUAL","partitions":[],"isComplete":false} | PARTIAL     | ["p_20231101_20231201"]                       | ["p_20231101_20231201"]                       | 100.00% (1/1) | 5b3b4525b6774b5b-89b070042cdcbcd5 |
| 509478131194211 | 23377 | inner_mtmv_23371 | 23371 | range_date_up_union_mv2  | 21805        | regression_test_nereids_rules_p0_mv_create_part_and_up | SUCCESS |          | 2025-01-08 18:19:10 | 2025-01-08 18:19:10 | 2025-01-08 18:19:10 | 156        | {"triggerMode":"SYSTEM","isComplete":false}                 | COMPLETE    | ["p_20231001_20231101"]                       | ["p_20231001_20231101"]                       | 100.00% (1/1) | 6d0a0782819b446e-b9da5d5de513ce00 |
| 509486057129101 | 23377 | inner_mtmv_23371 | 23371 | range_date_up_union_mv2  | 21805        | regression_test_nereids_rules_p0_mv_create_part_and_up | SUCCESS |          | 2025-01-08 18:19:17 | 2025-01-08 18:19:17 | 2025-01-08 18:19:18 | 213        | {"triggerMode":"MANUAL","partitions":[],"isComplete":false} | PARTIAL     | ["p_20231101_20231201"]                       | ["p_20231101_20231201"]                       | 100.00% (1/1) | f1303483e3db43e7-aa424acc32dc39ca |
| 509486143784554 | 23377 | inner_mtmv_23371 | 23371 | range_date_up_union_mv2  | 21805        | regression_test_nereids_rules_p0_mv_create_part_and_up | SUCCESS |          | 2025-01-08 18:19:18 | 2025-01-08 18:19:18 | 2025-01-08 18:19:18 | 151        | {"triggerMode":"MANUAL","partitions":[],"isComplete":false} | PARTIAL     | ["p_20231101_20231201"]                       | ["p_20231101_20231201"]                       | 100.00% (1/1) | 8d29b11ac41f4fe0-9d7c86372707310b |
| 488317385772600 | 21794 | inner_mtmv_21788 | 21788 | test_tablet_type_mtmv_mv | 16016        | zd                                                     | SUCCESS |          | 2025-01-08 12:26:29 | 2025-01-08 12:26:29 | 2025-01-08 12:26:29 | 1          | {"triggerMode":"MANUAL","partitions":[],"isComplete":true}  | NOT_REFRESH | []                                            | \N                                            | \N            |                                   |
| 437156301250803 | 19508 | inner_mtmv_19494 | 19494 | mv1                      | 16016        | zd                                                     | SUCCESS |          | 2025-01-07 22:13:48 | 2025-01-07 22:13:48 | 2025-01-07 22:17:45 | 236985     | {"triggerMode":"MANUAL","partitions":[],"isComplete":false} | COMPLETE    | ["p_20210101_MAXVALUE","p_20200101_20210101"] | ["p_20210101_MAXVALUE","p_20200101_20210101"] | 100.00% (2/2) | 7965b4ddce8a4480-8884e9701679c1c4 |
| 439689059641969 | 19508 | inner_mtmv_19494 | 19494 | mv1                      | 16016        | zd                                                     | SUCCESS |          | 2025-01-07 22:55:59 | 2025-01-07 22:55:59 | 2025-01-07 22:55:59 | 35         | {"triggerMode":"MANUAL","partitions":[],"isComplete":false} | NOT_REFRESH | []                                            | \N                                            | \N            |                                   |
+-----------------+-------+------------------+-------+--------------------------+--------------+--------------------------------------------------------+---------+----------+---------------------+---------------------+---------------------+------------+-------------------------------------------------------------+-------------+-----------------------------------------------+-----------------------------------------------+---------------+-----------------------------------+
```

View tasks for all insert tasks

```sql
select * from tasks("type"="insert");
```
```text
+----------------+----------------+----------------+-------------------------------+---------+----------+---------------------+---------------------+---------------------+-------------+---------------+------+
| TaskId         | JobId          | JobName        | Label                         | Status  | ErrorMsg | CreateTime          | StartTime           | FinishTime          | TrackingUrl | LoadStatistic | User |
+----------------+----------------+----------------+-------------------------------+---------+----------+---------------------+---------------------+---------------------+-------------+---------------+------+
| 79133848479750 | 78533940810334 | insert_tab_job | 78533940810334_79133848479750 | SUCCESS |          | 2025-01-17 14:42:54 | 2025-01-17 14:42:54 | 2025-01-17 14:42:54 |             |               | root |
+----------------+----------------+----------------+-------------------------------+---------+----------+---------------------+---------------------+---------------------+-------------+---------------+------+
```

