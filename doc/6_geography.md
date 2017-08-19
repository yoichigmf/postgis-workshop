6 - Geography
=============

作成
--------

経度 / 緯度テキストデータからgeographyを作成します 

```SQL
SELECT ST_SetSRID(ST_MakePoint(longitude_wgs84, latitude_wgs84), 4326)::geography AS geog
FROM rff.station;
```

同じものをQGIS DBManagerで見れるようにします:


```SQL
SELECT (row_number() OVER ()) AS gid,
       (ST_SetSRID(ST_MakePoint(longitude_wgs84, latitude_wgs84), 4326)::geography)::geometry AS geom
FROM rff.station;
```

このqueryについて :
- geometry vs geography
- geography は EPSG:4326 でのみ利用可能
- cast 
- DB Manager では (まだ)geography を正しく扱えません

距離
--------

geography間の（強引な）距離計算

```
WITH station AS (
  SELECT  row_number() OVER() AS id,
	  nom, dept,
          ST_SetSRID(ST_MakePoint(longitude_wgs84, latitude_wgs84), 4326)::geography AS geog
  FROM rff.station
  WHERE dept = 77
)

SELECT s1.nom, 
       s2.nom, 
       ST_Distance(s1.geog, s2.geog)

FROM   station AS s1, 
       station AS s2 

WHERE  s1.id > s2.id;  -- dist(a,b) と dist(b,a)の計算を除去するため
```


この query について:
- geographyで距離と計測機能が使えます
- geographyへのキャストを行わない場合の距離を見て比べてみてください


