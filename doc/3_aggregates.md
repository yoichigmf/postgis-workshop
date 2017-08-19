3 - 集約
==============

union
-----

地域内の区域を都市別に集約します

```SQL
SELECT code_reg,
       ST_Union(geom) AS geom
FROM admin.commune
GROUP BY code_reg;
```

QGIS DBManagerでデータをロードするためにユニークな整数のIDを追加します :

```SQL
SELECT (row_number() OVER ())::integer AS gid, 
       code_reg,
       ST_Union(geom) AS geom
FROM admin.commune
GROUP BY code_reg;
```


クエリについての解説 : 
- http://www.postgis.net/docs/manual-2.0/ST_Union.html
- row_number() と SQL WINDOWING

Collect
-------

前記のものと同じ,しかしここでは _ST_Union_ の代わりに _ST_Collect_ を使っています

```SQL
SELECT (row_number() OVER ())::integer AS gid, 
       ST_CollectionExtract(ST_Collect(geom), 3) AS geom
FROM admin.commune
GROUP BY code_reg;
```

クエリについての解説 : 
- MULTI* とは何でしょう？, COLLECTION とは何でしょう？
- 何故 ST_Collect は ST_Union より高速なのか
- http://www.postgis.net/docs/manual-2.0/ST_CollectionExtract.html

さらに collections について
----------------

前と似ていますがここでは _ST_CollectionExtract_ の代わりに  _ST_CollectionHomogenize_ を使っています

```SQL
SELECT (row_number() OVER ())::integer AS gid, 
       ST_CollectionHomogenize(ST_Collect(geom)) AS geom
FROM admin.commune
GROUP BY code_reg;
```

クエリについての解説 : 
- http://www.postgis.net/docs/manual-2.0/ST_CollectionHomogenize.html

Merge lines
-----------

Retrieve a single river (Rhone) and aggregate all LINESTRING subparts in a single geometry

_ST_LineMerge_ interest :

```SQL
SELECT
   ST_Summary(ST_LineMerge(ST_CollectionExtract(ST_Collect(geom), 2))) AS geom,
   ST_Summary(ST_CollectionExtract(ST_Collect(geom), 2)) AS geom 
FROM hydro.cours_eau 
WHERE code_hydro = 'V---0000';
-- Hydrological code from Rhone river
;
```

Using _ST_LineMerge_ and view it through QGIS DBManager:

```SQL
SELECT 1::integer AS gid,
      ST_LineMerge(ST_CollectionExtract(ST_Collect(geom), 2)) AS geom
FROM hydro.cours_eau 
-- Hydrological code from Rhone river
WHERE code_hydro = 'V---0000';
;
```
