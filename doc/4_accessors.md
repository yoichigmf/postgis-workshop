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

川のラインストリングから1個づつの点の情報を取得したい場合はループを使う必要があります

_generate_series(i, j)_ を使います:
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

query について:
- generate_series ループ
- 前のものと同じですが各点に対してループしています

これで点を取得できます :

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

最後にクエリをQGIS と DBManagerで表示できるようにします:

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

簡単に変換できる形式 : GeoJSON
---------------------------------------

ジオメトリをGeoJSON形式にして出力してみます 

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

こういう GeoJSON での出力もできます :

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

このqueryについて :
- http://www.postgis.net/docs/manual-2.0/ST_AsGeoJSON.html
- 使用する10進数の桁数

練習問題 :
- 結果をここでテストしてみましょう : http://geojsonlint.com/
- OpenLayers demonstration GeoJSON の入力にして表示してみましょう (ST_AsGeoJSON documentation pageにリンクがあります)
- GitHubで表示してみましょう : https://github.com/vpicavet/geojson-samples/blob/master/rhone.geojson

