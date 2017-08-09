2 - 基本的な解析
==================

人口密度が高い都市を探します
---------------------

```SQL

-- Retrieve the 100 most dense cities.

SELECT * 
FROM admin.commune 
ORDER BY population / ST_Area(geom) DESC
LIMIT 100;
```


演習 : 
- QGIS の DbManager プラグインでこの結果を見てみましょう

Toulouse の近所
-------------------

Toulouseという名前の都市の周りの都市の一覧を取得しましょう.

```SQL

SELECT c.gid, c.geom, c.nom_com
FROM admin.commune AS c
WHERE ST_Touches( c.geom, 
                  (SELECT geom FROM admin.commune WHERE nom_com = 'TOULOUSE')
                )
;
```
演習 : 
- QGIS の DbManager プラグインでこの結果を見てみましょう

最も長い河川
--------------

どの河川が最も長いでしょうか ?
 
```SQL

SELECT SUM(ST_Length(geom)), toponyme
FROM hydro.cours_eau
WHERE toponyme IS NOT NULL
GROUP BY toponyme
ORDER BY SUM(ST_Length(geom)) DESC
LIMIT 10;
```

注: 
- 最初の結果はデータが不完全なことに関係しています

queryの注意点 :
- NULL / NOT NULL
- GROUP BY and aggregate function as SUM
- ORDER BY ASC / DESC
- LIMIT

バッファ
-------

```SQL

SELECT ST_Buffer(geom, 10000) AS geom, gid from admin.commune where nom_com = 'TOULOUSE';

```
Make some more buffers around some cities. Then around the cities touching Toulouse.

Distances
---------

-- How far are big cities from Toulouse ?

```SQL

SELECT nom_com,
       population AS population,
       ST_Distance(geom, (SELECT geom FROM admin.commune WHERE nom_com = 'TOULOUSE')) / 1000 AS dist_km
FROM admin.commune
WHERE population > 150000
AND NOT nom_com = 'TOULOUSE';
```

About the query :
- SQL Sub Query
- ST_Distance
