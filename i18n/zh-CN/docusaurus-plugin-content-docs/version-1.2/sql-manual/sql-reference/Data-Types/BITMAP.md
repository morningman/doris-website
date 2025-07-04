---
{
    "title": "BITMAP",
    "language": "zh-CN"
}
---

## BITMAP
## 描述
    BITMAP
    BITMAP不能作为key列使用，建表时配合聚合类型为BITMAP_UNION。
    用户不需要指定长度和默认值。长度根据数据的聚合程度系统内控制。
    并且BITMAP列只能通过配套的bitmap_union_count、bitmap_union、bitmap_hash、bitmap_hash64等函数进行查询或使用。
    
    离线场景下使用BITMAP会影响导入速度，在数据量大的情况下查询速度会慢于HLL，并优于Count Distinct。
    注意：实时场景下BITMAP如果不使用全局字典，使用了bitmap_hash()可能会导致有千分之一左右的误差。如果这个误差不可接受，可以使用bitmap_hash64。

## 举例

建表示例如下：

    create table metric_table (
      datekey int,
      hour int,
      device_id bitmap BITMAP_UNION
    )
    aggregate key (datekey, hour)
    distributed by hash(datekey, hour) buckets 1
    properties(
      "replication_num" = "1"
    );

插入数据示例：

    insert into metric_table values
    (20200622, 1, to_bitmap(243)),
    (20200622, 2, bitmap_from_array([1,2,3,4,5,434543])),
    (20200622, 3, to_bitmap(287667876573));

查询数据示例：

    select hour, BITMAP_UNION_COUNT(pv) over(order by hour) uv from(
       select hour, BITMAP_UNION(device_id) as pv
       from metric_table -- 查询每小时的累计UV
       where datekey=20200622
    group by hour order by 1
    ) final;

在查询时，BITMAP 可配合`return_object_data_as_binary`变量进行使用，详情可查看[变量](../../../advanced/variables.md)章节。

### keywords

    BITMAP
