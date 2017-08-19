5 - ジオメトリの作成
=========================

ジオメトリキャスト
-------------

簡単な geometry キャストでジオメトリを作ります

```SQL
SELECT ST_AsText('POINT(10 10)'::geometry);
```


この query について:
- cast
- OGC WKT 文法

WKT + SRID
----------

前と同じですがSRIDを指定しています

```SQL
SELECT ST_AsEWKT('SRID=4326;POINT(10 10)'::geometry);
```

同じですが別の文法です
```SQL
SELECT ST_AsEWKT(ST_SetSRID('POINT(10 10)'::geometry, 4326));
```


このqueryについて :
- SRID
- http://www.postgis.net/docs/manual-2.0/ST_SRID.html
- http://www.postgis.net/docs/manual-2.0/ST_SetSRID.html

座標をジオメトリにします
-------------------

経度 / 緯度のテキストデータからジオメトリを作ります 
 
```SQL
SELECT ST_SetSRID(ST_MakePoint(longitude_wgs84, latitude_wgs84), 4326) AS geom
FROM rff.station;
```

同じですが QGIS DBManagerで見ることができるようにします:

```SQL
SELECT (row_number() OVER ()) AS gid,
       ST_SetSRID(ST_MakePoint(longitude_wgs84, latitude_wgs84), 4326) AS geom
FROM rff.station;
```

このqueryについて :
- http://www.postgis.net/docs/manual-2.0/ST_MakePoint.html
- http://www.postgis.net/docs/manual-2.0/ST_SetSRID.html
