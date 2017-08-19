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

���̌���
-----------

1�̐�̃f�[�^(Rhone)��ǂݏo���� ���ׂĂ� LINESTRING ��P��̃W�I���g���̃T�u�p�[�g�Ƃ��ďW�񂵂܂�

_ST_LineMerge_ �ɒ��ڂ��ĉ����� :

```SQL
SELECT
   ST_Summary(ST_LineMerge(ST_CollectionExtract(ST_Collect(geom), 2))) AS geom,
   ST_Summary(ST_CollectionExtract(ST_Collect(geom), 2)) AS geom 
FROM hydro.cours_eau 
WHERE code_hydro = 'V---0000';
-- Rhone��̐����R�[�h
;
```

_ST_LineMerge_ �𗘗p����QGIS DBManager�Ō��ʂ����Ă݂܂��傤:

```SQL
SELECT 1::integer AS gid,
      ST_LineMerge(ST_CollectionExtract(ST_Collect(geom), 2)) AS geom
FROM hydro.cours_eau 
-- Rhone��̐����R�[�h
WHERE code_hydro = 'V---0000';
;
```
