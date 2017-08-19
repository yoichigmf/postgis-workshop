5 - �W�I���g���̍쐬
=========================

�W�I���g���L���X�g
-------------

�ȒP�� geometry �L���X�g�ŃW�I���g�������܂�

```SQL
SELECT ST_AsText('POINT(10 10)'::geometry);
```


���� query �ɂ���:
- cast
- OGC WKT ���@

WKT + SRID
----------

�O�Ɠ����ł���SRID���w�肵�Ă��܂�

```SQL
SELECT ST_AsEWKT('SRID=4326;POINT(10 10)'::geometry);
```

�����ł����ʂ̕��@�ł�
```SQL
SELECT ST_AsEWKT(ST_SetSRID('POINT(10 10)'::geometry, 4326));
```


����query�ɂ��� :
- SRID
- http://www.postgis.net/docs/manual-2.0/ST_SRID.html
- http://www.postgis.net/docs/manual-2.0/ST_SetSRID.html

���W���W�I���g���ɂ��܂�
-------------------

�o�x / �ܓx�̃e�L�X�g�f�[�^����W�I���g�������܂� 
 
```SQL
SELECT ST_SetSRID(ST_MakePoint(longitude_wgs84, latitude_wgs84), 4326) AS geom
FROM rff.station;
```

�����ł��� QGIS DBManager�Ō��邱�Ƃ��ł���悤�ɂ��܂�:

```SQL
SELECT (row_number() OVER ()) AS gid,
       ST_SetSRID(ST_MakePoint(longitude_wgs84, latitude_wgs84), 4326) AS geom
FROM rff.station;
```

����query�ɂ��� :
- http://www.postgis.net/docs/manual-2.0/ST_MakePoint.html
- http://www.postgis.net/docs/manual-2.0/ST_SetSRID.html
