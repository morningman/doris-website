---
{
    "title": "ST_AZIMUTH",
    "language": "zh-CN"
}
---

## ST_Azimuth

### Syntax

`DOUBLE ST_Azimuth(GEOPOINT point1, GEOPOINT point2)`

## 描述

输入两个点，并返回由点 1 和点 2 形成的线段的方位角。方位角是点 1 的真北方向线与点 1 和点 2 形成的线段之间的角的弧度。

正角在球面上按顺时针方向测量。 例如，线段的方位角：

* 指北是 0
* 指东是 PI/2
* 指南是 PI
* 指西是 3PI/2

ST_Azimuth 存在以下边缘情况：

* 如果两个输入点相同，则返回 NULL。
* 如果两个输入点是完全对映点，则返回 NULL。
* 如果任一输入地理位置不是单点或为空地理位置，则会抛出错误。

## 举例

```
mysql> SELECT st_azimuth(ST_Point(1, 0),ST_Point(0, 0));
+----------------------------------------------------+
| st_azimuth(st_point(1.0, 0.0), st_point(0.0, 0.0)) |
+----------------------------------------------------+
|                                   4.71238898038469 |
+----------------------------------------------------+
1 row in set (0.03 sec)

mysql> SELECT st_azimuth(ST_Point(0, 0),ST_Point(1, 0));
+----------------------------------------------------+
| st_azimuth(st_point(0.0, 0.0), st_point(1.0, 0.0)) |
+----------------------------------------------------+
|                                 1.5707963267948966 |
+----------------------------------------------------+
1 row in set (0.01 sec)

mysql> SELECT st_azimuth(ST_Point(0, 0),ST_Point(0, 1));
+----------------------------------------------------+
| st_azimuth(st_point(0.0, 0.0), st_point(0.0, 1.0)) |
+----------------------------------------------------+
|                                                  0 |
+----------------------------------------------------+
1 row in set (0.01 sec)

mysql> SELECT st_azimuth(ST_Point(-30, 0),ST_Point(150, 0));
+--------------------------------------------------------+
| st_azimuth(st_point(-30.0, 0.0), st_point(150.0, 0.0)) |
+--------------------------------------------------------+
|                                                   NULL |
+--------------------------------------------------------+
1 row in set (0.02 sec)

```
### keywords
ST_AZIMUTH,ST,AZIMUTH
