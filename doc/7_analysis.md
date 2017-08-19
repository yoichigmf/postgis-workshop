7 - ���
============

Intersection
------------

1�{�̐�ƌ�������s���E�|���S�����ŏ�����Ō�܂ł̏��ԂŎ擾���܂�

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

�I���U�t���C�ē��e������QGIS DB Manager�ŕ\�����܂�

```SQL
SELECT  (row_number() OVER ())::integer AS gid,
        ST_Transform(ST_SetSRID(
            ST_MakePoint(longitude_wgs84, latitude_wgs84), 4326), 2154
        ) AS geom
FROM rff.station;
```


����query�ɂ��� :
- ��ԎQ��
- http://www.postgis.net/docs/manual-2.0/ST_Transform.html

���`
--------------

Generalize data : �W�I���g���̊ȗ���.

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

���K :
- QGIS DB Manager�Ō��Ă݂܂��傤
- ���ꂩ��P�����p�����[�^�������ς��Ă݂܂��傤

���� query �ɂ���:
- ���`
- �g�|���W�̖��
- http://www.postgis.net/docs/manual-2.0/ST_Simplify.html

�o�b�t�@�̌���
-------------

Toulouse �s�̒��S���� 50km �ȓ��ɂ���s���|���S�����擾���Ă݂�

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


�������ʂ� CTE���g��Ȃ��ꍇ, �܂��p�t�H�[�}���X�������ł�:

```SQL
SELECT gid, nom_com, geom
FROM admin.commune
WHERE ST_Intersects(geom,
 (SELECT ST_Buffer(ST_Centroid(geom), 50000)
  FROM admin.commune
  WHERE nom_com = 'TOULOUSE'
 ));
```

�������ʂŃp�t�H�[�}���X���悭�Ȃ���,���̂Ȃ� ST_Intersects �����d�ɌĂ΂�Ă��邩��ł�

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

�Ō�ł���������Ƃ���, _ST_Dwithin_ �@�\�̗��p :
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


���� query �ɂ���:
- ��ԃC���f�b�N�X�Ƌ�ԉ��Z�̃p�t�H�[�}���X�ɂ��Ă̍l�@

�ŋߖT�T��
-----------------

�傫�ȓs�s�͒ʏ�傫�Ȑ�̋߂��ɂ���܂�. ���ꂼ��̑�s�s�ɍł��߂�����������܂��傤

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

����query�ɂ��� :
- Bbox �� geometry
- �������Z�q &&
- KNN (k-Nearest Neighbor)���Z�q <#> and <->
