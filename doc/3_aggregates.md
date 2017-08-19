3 - �W��
==============

union
-----

�n����̋���s�s�ʂɏW�񂵂܂�

```SQL
SELECT code_reg,
       ST_Union(geom) AS geom
FROM admin.commune
GROUP BY code_reg;
```

QGIS DBManager�Ńf�[�^�����[�h���邽�߂Ƀ��j�[�N�Ȑ�����ID��ǉ����܂� :

```SQL
SELECT (row_number() OVER ())::integer AS gid, 
       code_reg,
       ST_Union(geom) AS geom
FROM admin.commune
GROUP BY code_reg;
```


�N�G���ɂ��Ẳ�� : 
- http://www.postgis.net/docs/manual-2.0/ST_Union.html
- row_number() �� SQL WINDOWING

Collect
-------

�O�L�̂��̂Ɠ���,�����������ł� _ST_Union_ �̑���� _ST_Collect_ ���g���Ă��܂�

```SQL
SELECT (row_number() OVER ())::integer AS gid, 
       ST_CollectionExtract(ST_Collect(geom), 3) AS geom
FROM admin.commune
GROUP BY code_reg;
```

�N�G���ɂ��Ẳ�� : 
- MULTI* �Ƃ͉��ł��傤�H, COLLECTION �Ƃ͉��ł��傤�H
- ���� ST_Collect �� ST_Union ��荂���Ȃ̂�
- http://www.postgis.net/docs/manual-2.0/ST_CollectionExtract.html

����� collections �ɂ���
----------------

�O�Ǝ��Ă��܂��������ł� _ST_CollectionExtract_ �̑����  _ST_CollectionHomogenize_ ���g���Ă��܂�

```SQL
SELECT (row_number() OVER ())::integer AS gid, 
       ST_CollectionHomogenize(ST_Collect(geom)) AS geom
FROM admin.commune
GROUP BY code_reg;
```

�N�G���ɂ��Ẳ�� : 
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
