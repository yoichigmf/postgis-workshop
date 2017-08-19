4 - ジオメトリアクセッサ
======================

バイナリ形式
-------------

PostGISに格納されている形式でジオメトリを読み出します

```SQL
SELECT geom 
FROM admin.commune
LIMIT 10;
```


queryについて :
- 16進数で表されたバイナリストレージ

読める情報
--------------

ジオメトリの構造を読み出します

```SQL

SELECT ST_Summary(geom) 
FROM admin.commune
LIMIT 10;
```

TIP: 
- PGAdmin ではマルチラインを表示できません, psqlを利用, copy-paste, または strings 関数を使って下さい

データの表示
------------

そろそろ人間が読める形式でデータを表示したいです 

```SQL
SELECT ST_AsText(geom) 
FROM admin.commune
LIMIT 10;
```

TIP : 
- データがとても大きい場合PGAdminでは何も表示できません (psql を代わりに使って下さい)

データアクセス
-----------

```SQL
-- 川を集約します 
-- ジオメトリのポイント数を読み出します
-- 特定のラインストリングの中の点を読み出します


WITH rhone AS 
(
  SELECT ST_LineMerge(ST_CollectionExtract(ST_Collect(geom), 2)) AS geom 
  FROM hydro.cours_eau 
  WHERE code_hydro = 'V---0000'  -- Hydrological code from Rhone river
)


SELECT ST_Summary(geom), 
       ST_NumPoints(geom), 
       ST_AsText(ST_PointN(geom, 55)) AS p_55

FROM rhone;
```

queryについて :
- SQL CTEとは何でしょう？
- http://www.postgis.net/docs/manual-2.0/ST_Summary.html
- http://www.postgis.net/docs/manual-2.0/ST_NumPoints.html
- http://www.postgis.net/docs/manual-2.0/ST_PointN.html

全ポイントの取得
--------------

If we want to retrieve every single point for the river linestring, we need to perform a loop

We will use _generate_series(i, j)_ :
```SQL
select generate_series(1, 55);

select * from generate_series(1, 55);
```


```SQL
WITH rhone AS 
(
  SELECT ST_LineMerge(ST_CollectionExtract(ST_Collect(geom), 2)) AS geom 
  FROM hydro.cours_eau 
  WHERE code_hydro = 'V---0000'  -- Hydrological code from Rhone river
)
SELECT generate_series(1, ST_NumPoints(geom)) as n
FROM rhone;
```

About the query :
- generate_series loop
- Same as previously but with the loop on each point

Now get the points :

```SQL
WITH rhone AS 
(
  SELECT ST_LineMerge(ST_CollectionExtract(ST_Collect(geom), 2)) AS geom 
  FROM hydro.cours_eau 
  WHERE code_hydro = 'V---0000'  -- Hydrological code from Rhone river

), loop AS (
  SELECT generate_series(1, ST_NumPoints(geom)) AS n
  FROM rhone
)
SELECT n, ST_AsText(ST_PointN(geom, n))
FROM rhone, loop;

```

And finally the query displayable with QGIS and DBManager:

```SQL
WITH rhone AS 
(
  SELECT ST_LineMerge(ST_CollectionExtract(ST_Collect(geom), 2)) AS geom 
  FROM hydro.cours_eau 
  WHERE code_hydro = 'V---0000'  -- Hydrological code from Rhone river

), loop AS (
  SELECT generate_series(1, ST_NumPoints(geom)) AS n
  FROM rhone
)

SELECT n::integer AS gid, ST_PointN(geom, n) AS geom
FROM rhone, loop;
```

The little format that could : GeoJSON
---------------------------------------

Output a geometry to GeoJSON output format 

```SQL
WITH rhone AS 
(
  SELECT ST_LineMerge(ST_CollectionExtract(ST_Collect(geom), 2)) AS geom 
  FROM hydro.cours_eau 
  WHERE code_hydro = 'V---0000'  -- Hydrological code from Rhone river

)

SELECT ST_AsGeoJSON(ST_Transform(geom, 4326), 5) 
FROM rhone;
```

Some more GeoJSON output :

```SQL
select 
    ST_AsGeojson(ST_Collect(ST_Transform(geom, 4326)), 5) 
from (
    select 
        * 
    from 
        admin.commune 
    where 
        code_dept = '69'
) as coms;
```

About the query :
- http://www.postgis.net/docs/manual-2.0/ST_AsGeoJSON.html
- number of decimal digit to use

Practice :
- Test the result : http://geojsonlint.com/
- Display it in OpenLayers demonstration GeoJSON input (link in ST_AsGeoJSON documentation page)
- Display it on GitHub : https://github.com/vpicavet/geojson-samples/blob/master/rhone.geojson

