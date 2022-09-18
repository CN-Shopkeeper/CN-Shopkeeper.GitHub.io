---
layout: post
author: shopkeeper
categories: gis
---

# pgRouting

[参考资料](https://blog.csdn.net/sinat_41310868/article/details/106912152)

Windows版本的PostGIS已经捆绑安装了pgRouting，只需要在数据库拓展插件即可。

```sql
CREATE EXTENSION PostGIS
CREATE EXTENSION pgRouting
```

## 创建边的表

```sql
CREATE TABLE edge_table (
    id BIGSERIAL,
    dir VARCHAR,
    source BIGINT,
    target BIGINT,
    cost FLOAT,
    reverse_cost FLOAT,
    capacity BIGINT,
    reverse_capacity BIGINT,
    category_id INTEGER,
    reverse_category_id INTEGER,
    x1 FLOAT,
    y1 FLOAT,
    x2 FLOAT,
    y2 FLOAT,
    the_geom geometry
);
```

其中id为线的id；source、target为边的起点和终点的id；cost、reverse_cost为正向、反向的开销，如果为-1则不可达。这五个参数依次是pgRouting的迪杰斯特拉函数的五个输入列。

x1, y1, x2, y2为起点终点的坐标。the_geom存贮最后建立的线。

## 插入边的数据

```sql
INSERT INTO edge_table (
    category_id, reverse_category_id,
    cost, reverse_cost,
    capacity, reverse_capacity,
    x1, y1,
    x2, y2) VALUES
(3, 1,    1,  1,  80, 130,   2,   0,    2, 1),
(3, 2,   -1,  1,  -1, 100,   2,   1,    3, 1),
(2, 1,   -1,  1,  -1, 130,   3,   1,    4, 1),
(2, 4,    1,  1, 100,  50,   2,   1,    2, 2),
(1, 4,    1, -1, 130,  -1,   3,   1,    3, 2),
(4, 2,    1,  1,  50, 100,   0,   2,    1, 2),
(4, 1,    1,  1,  50, 130,   1,   2,    2, 2),
(2, 1,    1,  1, 100, 130,   2,   2,    3, 2),
(1, 3,    1,  1, 130,  80,   3,   2,    4, 2),
(1, 4,    1,  1, 130,  50,   2,   2,    2, 3),
(1, 2,    1, -1, 130,  -1,   3,   2,    3, 3),
(2, 3,    1, -1, 100,  -1,   2,   3,    3, 3),
(2, 4,    1, -1, 100,  -1,   3,   3,    4, 3),
(3, 1,    1,  1,  80, 130,   2,   3,    2, 4),
(3, 4,    1,  1,  80,  50,   4,   2,    4, 3),
(3, 3,    1,  1,  80,  80,   4,   1,    4, 2),
(1, 2,    1,  1, 130, 100,   0.5, 3.5,  1.999999999999,3.5),
(4, 1,    1,  1,  50, 130,   3.5, 2.3,  3.5,4);
```

## 建立线与连通性
```sql
UPDATE edge_table SET the_geom = st_makeline(st_point(x1,y1),st_point(x2,y2)),
dir = CASE WHEN (cost>0 AND reverse_cost>0) THEN 'B'   -- both ways，双向通行
           WHEN (cost>0 AND reverse_cost<0) THEN 'FT'  -- direction of the LINESSTRING，沿路通行
           WHEN (cost<0 AND reverse_cost>0) THEN 'TF'  -- reverse direction of the LINESTRING，反向通行
           ELSE '' END;                                -- unknown，未知
```

## 拓扑
```sql
SELECT pgr_createTopology('edge_table',0.001);
```

用于构建网络拓扑，会生成表edge_table_vertices_pgr

参数说明：

```
edge_table：text，表名

tolerance：float8，误差缓冲值，两个点的距离在这个距离内，就算重合为一点。这个距离使用st_length计算
```

## 兴趣点建表

本篇并不涉及兴趣点的路径规划，将在第二部分记录。

```sql
CREATE TABLE pointsOfInterest(
    pid BIGSERIAL,
    x FLOAT,
    y FLOAT,
    edge_id BIGINT,
    side CHAR,
    fraction FLOAT,
    the_geom geometry,
    newPoint geometry
);
```

## 插入兴趣点数据
```sql
INSERT INTO pointsOfInterest (x, y, edge_id, side, fraction) VALUES
(1.8, 0.4,   1, 'l', 0.4),
(4.2, 2.4,  15, 'r', 0.4),
(2.6, 3.2,  12, 'l', 0.6),
(0.3, 1.8,   6, 'r', 0.3),
(2.9, 1.8,   5, 'l', 0.8),
(2.2, 1.7,   4, 'b', 0.7);
```

## 生成point
```sql
UPDATE pointsOfInterest SET the_geom = st_makePoint(x,y);
```

## 生成临时点
```sql
UPDATE pointsOfInterest
    SET newPoint = ST_LineInterpolatePoint(e.the_geom, fraction)
    FROM edge_table AS e WHERE edge_id = id;
```

## 查询路径

```sql
SELECT * FROM pgr_dijkstra(   
 'SELECT id, source, target, cost, reverse_cost FROM edge_table',  
  2, 3);
```

第一个参数是要路径查询的表的查询语句；第二个参数是起点的id；第三个参数是终点的id。

注意，这里查询的起点和重点并不是兴趣点，而是之前拓扑生成的点。根据兴趣点查询路径将在第二部分记录。
