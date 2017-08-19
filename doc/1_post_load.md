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
- PostGIS 2.0では geometry_columns がビューになりました.これによって格納されたジオメトリデータとメタデータの( geometry_columnsからの)リレーションが切れるリスクが無くなりました .


データチェック
----------

不正なジオメトリがデータセットにないかチェックします

```SQL

SELECT gid, ST_IsValidReason(geom) 
FROM hydro.region 
WHERE NOT ST_IsValid(geom);

SELECT gid, ST_IsValidReason(geom) 
FROM admin.commune 
WHERE NOT ST_IsValid(geom);
```


注 : POINT データは常に正しい形式です, ここでは面データを対象にします (大部分のデータ不正は面データでおこります)


queryについて : 
- 不正なデータとはどういうものですか ?
- OGC SFS 仕様

インデックスのチェック
-------------

空間インデックスが正しく作られているかどうかチェックして下さい

PgAdmin または psql を立ち上げて \di を打ち込んで下さい

インデックスについて : 
- 空間インデックスとは何でしょうか?
- それは何の役にたつのでしょうか?
