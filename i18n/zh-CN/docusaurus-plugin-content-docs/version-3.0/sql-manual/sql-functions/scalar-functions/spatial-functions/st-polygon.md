---
{
    "title": "ST_POLYGON",
    "language": "zh-CN"
}
---

## 描述

将 WKT（Well-Known Text）格式的字符串 转换为内存中的 多边形（Polygon）几何对象

## 别名

- st_polygonfromtext
- st_polyfromtext

## 语法

```sql
ST_POLYGON( <wkt>)
```

## 参数

| 参数   | 说明                 |
|------|--------------------|
| `<wkt>` | 由 POLYGON 函数生成的一个多边形的wkt文本 |

## 返回值

返回 Polygon 类型的几何对象，该对象在内存中以 Doris 内部的空间数据格式存储，可直接作为参数传入其他空间函数（如 ST_AREA、ST_CONTAINS 等）进行计算

- 若输入的 WKT 字符串格式无效（如环未闭合、语法错误），返回 NULL。
- 若输入 <wkt> 为 NULL 或空字符串，返回 NULL。

## 举例

基本多边形
```sql
SELECT ST_AsText(ST_Polygon("POLYGON ((0 0, 10 0, 10 10, 0 10, 0 0))"));
```

```text
+------------------------------------------------------------------+
| st_astext(st_polygon('POLYGON ((0 0, 10 0, 10 10, 0 10, 0 0))')) |
+------------------------------------------------------------------+
| POLYGON ((0 0, 10 0, 10 10, 0 10, 0 0))                          |
+------------------------------------------------------------------+
```

自相交多边形(无效)

```sql
mysql> select st_polygon('POLYGON ((0 0, 1 1, 0 1, 1 0, 0 0))');
+---------------------------------------------------+
| st_polygon('POLYGON ((0 0, 1 1, 0 1, 1 0, 0 0))') |
+---------------------------------------------------+
| NULL                                              |
+---------------------------------------------------+
```

无效 WKT（环未闭合）

```sql
mysql> SELECT ST_Polygon("POLYGON ((0 0, 10 0, 10 10, 0 10))");
+--------------------------------------------------+
| ST_Polygon("POLYGON ((0 0, 10 0, 10 10, 0 10))") |
+--------------------------------------------------+
| NULL                                             |
+--------------------------------------------------+
```

无效 WKT（语法错误）

```sql

mysql> SELECT ST_Polygon("POLYGON (0 0, 10 0, 10 10, 0 10, 0 0)");
+-----------------------------------------------------+
| ST_Polygon("POLYGON (0 0, 10 0, 10 10, 0 10, 0 0)") |
+-----------------------------------------------------+
| NULL                                                |
+-----------------------------------------------------+
```

输入 NULL

```sql
mysql> SELECT ST_Polygon(NULL);
+------------------+
| ST_Polygon(NULL) |
+------------------+
| NULL             |
+------------------+
```