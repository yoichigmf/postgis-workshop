6 - Geography
=============

�쐬
--------

�o�x / �ܓx�e�L�X�g�f�[�^����geography���쐬���܂� 

```SQL
SELECT ST_SetSRID(ST_MakePoint(longitude_wgs84, latitude_wgs84), 4326)::geography AS geog
FROM rff.station;
```

�������̂�QGIS DBManager�Ō����悤�ɂ��܂�:


```SQL
SELECT (row_number() OVER ()) AS gid,
       (ST_SetSRID(ST_MakePoint(longitude_wgs84, latitude_wgs84), 4326)::geography)::geometry AS geom
FROM rff.station;
```

����query�ɂ��� :
- geometry vs geography
- geography �� EPSG:4326 �ł̂ݗ��p�\
- cast 
- DB Manager �ł� (�܂�)geography �𐳂��������܂���

����
--------

geography�Ԃ́i�����ȁj�����v�Z

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

WHERE  s1.id > s2.id;  -- dist(a,b) �� dist(b,a)�̌v�Z���������邽��
```


���� query �ɂ���:
- geography�ŋ����ƌv���@�\���g���܂�
- geography�ւ̃L���X�g���s��Ȃ��ꍇ�̋��������Ĕ�ׂĂ݂Ă�������


