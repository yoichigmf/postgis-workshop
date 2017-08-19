PostGIS ワークショップ
================

リソース
---------

便利なリンク集 :
* PostGIS Home : http://www.postgis.net
* PostGIS Documentation : http://www.postgis.net/docs/manual-2.0/
* PostgreSQL Documention : http://www.postgresql.org/docs/9.2/interactive/index.html
* GitHubにあるこのワークショップ : https://github.com/vpicavet/postgis-workshop
* GeoJSON の検証 : http://geojsonlint.com/
* GeoJSON on GitHub : https://help.github.com/articles/mapping-geojson-files-on-github

始めましょう
-----------

最初にこのワークショップの最新版をダウンロードして下さい :

https://codeload.github.com/vpicavet/postgis-workshop/zip/master

ホームディレクトリに保存して _workshop_ ディレクトリに解凍して下さい.

```bash
$ cd ~
$ wget -O postgis-workshop-master.zip 'https://codeload.github.com/vpicavet/postgis-workshop/zip/master'
$ mkdir workshop
$ unzip -o postgis-workshop-master.zip -d workshop
$ cd workshop/postgis-workshop-master
```

最初のステップはデータベースの初期化とデータのロードです.

_scripts_ ディレクトリにある3個のスクリプトを動かしてデータベースの作成,データのダウンロード,データのデータベースへのロードを行ってください.

shapefilesをロードする場合 _shp2pgsql-gui_ GUI ツールを使うこともできます.

何か問題があった場合これらのスクリプトを再度実行することができます.
```bash
$ cd scripts
$ sudo -u postgres bash STEP_1-CREATE.sh
$ bash STEP_2-DOWNLOAD-DATA.sh
$ sudo -u postgres bash STEP_3-LOAD.sh
```

この時点で pgAdmin3 を使ってデータを見ることができます, またデータを QGISにロードすることができます.

 QGISをつかってデータに対する素敵な描画設定を試して下さい そして次のステップを実行して下さい.
