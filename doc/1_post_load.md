1 - Post-loading
================

PostGISのチェック
-------------

```SQL

-- カレントデータベースに 
-- エクステンションが正しくロードされているかどうかチェックします

SELECT postgis_full_version();
```

メタデータのチェック
--------------

```SQL
-- メタデータ情報が正しく設定されているかどうかチェックします

SELECT * FROM geometry_columns;
```


queryについて :
- With PostGIS 2.0 geometry_columns is now a view so no risk to decorrelate geometry data stored and metadata (from geometry_columns).


データチェック
----------

Check if any invalid geometry in a dataset

```SQL

SELECT gid, ST_IsValidReason(geom) 
FROM hydro.region 
WHERE NOT ST_IsValid(geom);

SELECT gid, ST_IsValidReason(geom) 
FROM admin.commune 
WHERE NOT ST_IsValid(geom);
```


NOTE : POINT data are always valid, we focus here on surfacic data  (far most invalid cases)


About the query : 
- What is invalid data ?
- OGC SFS specifications

Check indexes
-------------

Check if Spatial Indexes were correctly created

With PgAdmin or with psql and \di command

About indexes : 
- What is spatial index?
- What does it stand for?
