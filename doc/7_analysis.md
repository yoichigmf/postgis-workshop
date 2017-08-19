7 - 解析
============

Intersection
------------

1本の川と交差する行政界ポリゴンを最初から最後までの順番で取得します

```SQL
WITH river AS
(
  SELECT geom
  FROM hydro.cours_eau
  WHERE code_hydro = 'V---0000'  -- Hydrological code from Rhone river
)

SELECT nom_com
FROM admin.commune AS c, river AS r
WHERE ST_Intersects(c.geom, r.geom)
ORDER BY ST_LineLocatePoint(
    ST_LineMerge(r.geom),
    ST_ClosestPoint(r.geom, c.geom))
;
```

オンザフライ再投影をしてQGIS DB Managerで表示します

```SQL
SELECT  (row_number() OVER ())::integer AS gid,
        ST_Transform(ST_SetSRID(
            ST_MakePoint(longitude_wgs84, latitude_wgs84), 4326), 2154
        ) AS geom
FROM rff.station;
```


このqueryについて :
- 空間参照
- http://www.postgis.net/docs/manual-2.0/ST_Transform.html

総描
--------------

Generalize data : ジオメトリの簡略化.

```SQL
WITH river AS
(
  SELECT ST_LineMerge(ST_CollectionExtract(ST_Collect(geom), 2)) AS geom
  FROM hydro.cours_eau
  WHERE code_hydro = 'V---0000'  -- Hydrological code from Rhone river
)

SELECT 1::integer AS gid, ST_Simplify(geom, 250) AS geom
FROM river;
```

演習 :
- QGIS DB Managerで見てみましょう
- それから単純化パラメータを少し変えてみましょう

この query について:
- 総描
- トポロジの問題
- http://www.postgis.net/docs/manual-2.0/ST_Simplify.html

バッファの検索
-------------

Toulouse 市の中心から 50km 以内にある行政ポリゴンを取得してみる

```SQL
WITH toulouse_50km AS
(
  SELECT ST_Buffer(ST_Centroid(geom), 50000) AS geom
  FROM admin.commune
  WHERE nom_com = 'TOULOUSE'
)

SELECT gid, nom_com, c.geom
FROM admin.commune AS c, toulouse_50km AS t
WHERE ST_Intersects(c.geom, t.geom);
```


同じ結果で CTEを使わない場合, またパフォーマンスもいいです:

```SQL
SELECT gid, nom_com, geom
FROM admin.commune
WHERE ST_Intersects(geom,
 (SELECT ST_Buffer(ST_Centroid(geom), 50000)
  FROM admin.commune
  WHERE nom_com = 'TOULOUSE'
 ));
```

同じ結果でパフォーマンスがよくない例,何故なら ST_Intersects が多重に呼ばれているからです

```SQL
WITH toulouse AS
(
  SELECT ST_Centroid(geom) AS geom
  FROM admin.commune
  WHERE nom_com = 'TOULOUSE'
)

SELECT gid, nom_com, c.geom
FROM admin.commune AS c, toulouse AS t
WHERE ST_Intersects(c.geom, ST_Buffer(t.geom, 50000));
```

最後ですがちょっとした, _ST_Dwithin_ 機能の利用 :
```SQL
select
    t1.*
from
    admin.commune as t1
join
    admin.commune as t2
on
    ST_DWithin(t1.geom, ST_Centroid(t2.geom), 50000)
where
    t2.nom_com = 'TOULOUSE';
```


この query について:
- 空間インデックスと空間演算のパフォーマンスについての考察

最近傍探索
-----------------

大きな都市は通常大きな川の近くにあります. それぞれの大都市に最も近い川を検索しましょう

```SQL
SELECT c.nom_com,
       c.population * 1000 AS population,
       r.toponyme,
       ST_Distance(r.geom, c.geom) AS dist

FROM admin.commune AS c,
     hydro.cours_eau AS r

WHERE r.classe = '1' -- big river only
AND   r.toponyme IS NOT NULL
AND   (c.population * 1000) > 100000
AND   r.geom && ST_Expand(c.geom, 10000)
ORDER BY r.geom <-> c.geom;
```

このqueryについて :
- Bbox と geometry
- 交差演算子 &&
- KNN (k-Nearest Neighbor)演算子 <#> and <->
