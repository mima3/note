## 概要  
SpatiaLiteは地図などの空間情報を格納できるSQLiteの拡張です。  
ShpファイルやGeoJsonで記述されたGeometry情報をデータベースに格納し利用できます。  
  
http://www.gaia-gis.it/gaia-sins/index.html  
  
## 配布ファイルの説明  
### SpatiaLite  
コマンドラインからデータベースの操作を行えます。  
  
 **バイナリの配布**   
Windowsの場合は以下から任意のプラットフォームのバイナリが取得できます。  
http://www.gaia-gis.it/gaia-sins/index.html  
  
### mod_spatialite.dll/mod_spatialite.so  
SQLiteの拡張モジュールです。  
SQLite上で、load_extensionを利用してDLLまたはsoファイルを読み込むことでSpatiaLiteの機能が使用できるようになります。  
  
 **バイナリの配布**   
Windowsの場合は以下から任意のプラットフォームのバイナリが取得できます。  
http://www.gaia-gis.it/gaia-sins/index.html  
  
 **ソースコードの配布**   
以下からlibspatialiteのソースコードをダウンロードしてビルドすることで作成できます。  
https://www.gaia-gis.it/fossil/libspatialite/index  
  
### spatialite_gui  
GUIでspatialiteの機能が使用できます。  
![spatialite2.png](/image/fc86e7d9-ee70-7f29-1a63-bcf77d14ed37.png)  
  
POLYGONなどのGeometry型の列については、画像としてその内容を確認できます。  
![spatialite.png](/image/efc395eb-e88c-f9d4-846b-5e65205676e7.png)  
  
 **バイナリの配布**   
Windowsの場合は以下から任意のプラットフォームのバイナリが取得できます。  
http://www.gaia-gis.it/gaia-sins/index.html  
  
## libspatialiteのビルド  
もし、バイナリでmod_spatialiteを取得できない場合は、libspatialiteをビルドする必要があります。  
  
libspatialite-4.2.0のビルドには下記のライブラリが必要です。  
 **proj-4.8.0**  
proj.4は地図作成投影ライブラリです。  
http://trac.osgeo.org/proj/  
  
 **geos-3.4.2**   
Geometry EnginはJavaTopologySuiteのC++への移植で、Geometryを処理するための空間述語と機能のAPIです。  
http://trac.osgeo.org/geos/  
  
以上をインストールしたあとに、libsatialiteのソースがあるフォルダにて下記のコマンドを実行してください。  
  
```
./configure --disable-freexl
make
make install
```  
  
--disable-freexlでExcelとの連携機能を無効にしてます。  
これを有効にした場合はfreexlが必要になります。  
https://www.gaia-gis.it/fossil/freexl/index  
  
### FreeBSDでエラーが出る場合  
FreeBSDで下記のようなエラーがでる場合があります  
  
```
/usr/bin/ld: cannot find -ldl
```  
  
FreeBSDではdlopen は標準ライブラリなので-ldlは不要です。  
src/Makefileで-ldlを検索して削除したのちにmakeをし直しましょう。  
  
参考：  
http://sourceforge.net/p/idjc/discussion/458834/thread/246f0841/  
  
## mod_spatialiteをSQLiteから使用する。  
先に述べたように、load_extensionでspatialiteが利用できますが、いくつか注意点があります。  
  
・SQLiteのバージョンは3.7.17以降である必要があります。  
  
・依存するDLLはすべて、SQLiteから認識できる場所に配置される必要があります。関連のDLLのあるフォルダを環境変数PATHで指定してください。  
  
・SQLiteとmod_spatialiteのプロセスは同じビットでビルドしなければいけません。  
　SQLiteが32bitで動作している場合はmod_spatialiteは32bitでビルドする必要がり、SQLiteが64bitで動作している場合はmod_spatialiteは64bitでビルドする必要があります。  
　もし、Windowsで64bitバイナリのSQLiteが必要な場合は下記を記事通りに行えば作成できます。  
http://qiita.com/akaneko3/items/0e99c3c1366dfbad006f  
  
・load_extensionを利用してDLLまたはsoファイルを読み込むことで、以降spatialiteの機能が利用できるようになります。  
  
```sql
SELECT load_extension('/usr/local/lib/mod_spatialite.so');
```  
  
・SQLite3からデータベースを作成した場合、SpatiaLiteが管理するメタデータが作成されていません。このメタデータを作成するために下記のSQLを実行します。  
  
```sql
SELECT InitSpatialMetaData();
```  
  
## チュートリアル  
ここでは、実際に国土数値情報の行政区域をspatialiteに格納して利用してみます。  
  
### 国土数値情報で全国の行政区域を取得する。  
まず下記のページから全国の行政区域をダウンロードします。  
http://nlftp.mlit.go.jp/ksj/gml/datalist/KsjTmplt-N03.html  
  
### 空のSQLiteファイルを作成する。  
spatialite_guiを起動して空のSQLiteのデータベースファイルを作成します。  
  
[Creating a New (empty) SQLite DB]アイコンを押して、ファイル名を入力します。  
![spatialite001.png](/image/1bb96499-6ac4-5423-6aa0-f0e84b564722.png)  
  
正常に作成されると、作成されたデータベースへのパスが表示されます。  
![spatialite002.png](/image/a136324a-25b6-2deb-434a-4ecbc46c84cd.png)  
  
ここまでは普通のデータベースと変わりありません。  
  
### shpファイルをインポート  
国土数値情報のshpファイルをデータベースに取り込みます。  
  
[Load Shape file]アイコンを押して、国土数値情報の「N03-14_140401.shp」を選択します。  
  
![spatialite003.png](/image/0418627e-11a1-1db2-f549-0bc921aeac62.png)  
  
  
この時、エンコードを聞かれるので「CP932」を選択してください。  
![spatialite004.png](/image/c6c940b7-0f7b-ae66-3c04-34ecb160899b.png)  
  
正常に完了すると「N03-14_140401」というテーブルが作成されます。  
![spatialite005.png](/image/3dbc9d6f-5cb4-ff7d-15ca-efe7bf5c796a.png)  
  
### shpファイルから作成されたテーブルの確認  
ここではshpファイルから作成されたテーブルの確認を行います。  
作成されたテーブルの列を確認するにはテーブルを選択して、右クリックを押して「Show columns」を選択します。  
  
![spatialite006.png](/image/c5d78add-2fcd-e5d3-40f9-d2cdaeb2526c.png)  
  
![spatialite007.png](/image/954f783c-5542-0380-70ce-cf37f13a7960.png)  
  
N03_001～N03_004は県、区、市、N03_007にはコードが格納されているTEXT型の列です。  
Geometryは行政区域の形がBlobとして格納されていてPOLYGONというspatiaLiteが指定した型になっています。  
  
このテーブルの作成に実際しようされたSQL文はテーブルを右クリックして[Show CREATE statement]で確認できます。  
  
```sql
CREATE TABLE "N03-14_140401" (
"PK_UID" INTEGER PRIMARY KEY AUTOINCREMENT,
"N03_001" TEXT,
"N03_002" TEXT,
"N03_003" TEXT,
"N03_004" TEXT,
"N03_007" TEXT, 
"Geometry" POLYGON)
```  
  
では実際にテーブルの内容を見てみます。  
そのために、SQL文を実行します。  
  
```sql
SELECT * FROM "N03-14_140401" LIMIT 10;
```  
  
![spatialite008.png](/image/1c503d9e-da00-287a-14a3-bbb3d3cad9d2.png)  
  
実行すると、以下のような情報が取得できます。  
  
![spatialite009.png](/image/cb4294f1-3a0f-fcd6-5483-e86ef59b646d.png)  
  
PK_UID～N03_007はテキストでデータが表現されています。  
しかし、GeometryはBLOBデータなので人が見て内容が確認できません。  
  
この内容を確認する方法は２通りあります。  
１つはBLOB exploreを使用する方法です。  
Geometryの列の任意のセルを選択して右クリックを押し「BLOB explore」を選択します。  
  
![spatialite010.png](/image/58c8f516-bc86-51ba-103f-ff62362545f7.png)  
  
BLOB exploreではバイナリ、画像、SVG、GeoJSONなどの様々な形式でデータを表現することができます。  
![spatialite011.png](/image/52019b3b-360c-06fa-a0c7-e80f41c1e557.png)  
  
BLOB exploreを使用しない方法としては、SQLでGeometryの列を任意の形式で表示する方法があります。  
  
以下の例ではテキスト形式でGeometry列を表示しています。  
  
```sql
-- データが長いので最初の100文字まで表示
SELECT substr(ASTEXT(Geometry), 1, 100) FROM "N03-14_140401" LIMIT 10;
```  
  
![spatialite012.png](/image/0cedd1d4-b542-a05c-e971-8cee14033e94.png)  
  
ASTEXTでPOLYGON情報を文字として表示できました。これは任意の形式で出力することが可能です。たとえば、ASGEOJSONとした場合は、GeoJSONとして出力されます。  
  
```sql
SELECT substr(ASGEOJSON(Geometry), 1, 100) FROM "N03-14_140401" LIMIT 10;
```  
  
![spatialite013.png](/image/aa019d39-461d-861b-83cb-58ea72f5a533.png)  
  
ASTEXTやASGEOJSONのSpatiaLiteの提供する関数は下記の「Spatial SQL functions reference guide」に記載されています。  
  
http://www.gaia-gis.it/gaia-sins/spatialite-sql-4.2.0.html  
  
### 自分で座標情報を持つテーブルを作成してみる  
先の例ではshpファイルからテーブルを作りましたが、今回は、座標情報を持つテーブルを自分で作ってみます。  
  
以下のテーブルでは東京タワーなどの地物情報の座標を持つテーブルを作成します。  
  
まず、地物情報を含まない列のみを作成します。  
  
```sql
CREATE TABLE "places" (
"PK_UID" INTEGER PRIMARY KEY AUTOINCREMENT,
"name" TEXT)
```  
  
SQLを実行した後に、refreshを行うとテーブルが追加されます。  
  
![spatialite014.png](/image/32608abf-7e38-e535-21a7-e4b11145cd5b.png)  
  
次に、地物情報を表す列をAddGeometryColumn()を用いて作成します。  
  
```sql
Select AddGeometryColumn ('places', 'Geometry', 0, 'POINT', 'XY')
```  
  
SQLを実行した後に、refreshを行うと列が追加されて、テーブルに地球のアイコンがつきます。  
  
![sptialite100.png](/image/45d8f832-399b-c4e4-62da-883c73c145ec.png)  
  
  
テーブルを作成できたら、次はデータを入力します。  
以下の例では都庁の座標をテーブルに格納しています。  
  
```sql
INSERT INTO places (name, Geometry) VALUES(
  '都庁',
  GeomFromText('POINT(139.692101 35.689634 )')
)
```  
  
GeomFromTextは文字列をGeometryに変更する関数です。今回はテキスト形式から変換しましたが、GeomFromGeoJSON、GeomFromKmlなどを利用してGeoJSONやKmlから作成することができます。  
  
  
なお、以下のように一気に作ると、一見それっぽく動作しますが、メタデータが関連付けられていないので、RTreeインデックスを作るときなどに正常に動作しません。  
  
```sql
CREATE TABLE "places" (
"PK_UID" INTEGER PRIMARY KEY AUTOINCREMENT,
"name" TEXT,
"Geometry" POINT)
```  
  
  
  
### 座標情報に対する検索  
ここでは座標情報について検索します。  
placesテーブルで指定した座標が、どの行政区画に含まれているか検索してみます。  
  
```sql
SELECT
  name, "N03-14_140401".*
FROM
 "N03-14_140401"
INNER JOIN places ON
 Contains("N03-14_140401".Geometry, places.Geometry)
```  
  
Contains関数は第一引数の範囲に第二引数の領域が含まれているか調べます。  
この結果は次のようになります。  
  
![spatialite015.png](/image/009ed694-9481-cda0-9be6-bb279248c26d.png)  
  
都庁は新宿区にあるので期待どおりの結果といえるでしょう。  
先の例ではテーブルに格納されているデータで検索しましたが、直接WHERE区に指定することも可能です。  
  
  
```sql
SELECT
  *
FROM
 "N03-14_140401"
WHERE
  Contains("N03-14_140401".Geometry,GeomFromText('POINT(139.692101 35.689634 )'))
```  
  
### MBRと大まかな検索  
先ほどの検索は各座標について正確に検索していましたが、大まかな位置を早く取得したい場合もあります。  
  
この場合は、Minimum Bounding Rectangle - MBRを使用します。  
MBRはGEOMETRYのおおまかな大きさを表すものです。  
  
以下のような複雑な図形を赤い枠線の図形とみなします。  
  
![spatialite016.png](/image/bec3dd45-0c8c-12ea-5061-ba571c8aac2f.png)  
  
これにより、不正確ではあるが、早い検索が行えます。  
GEOMETRYからMBRを取得するには Envelope()を用います。  
  
```sql
SELECT
   ASTEXT(Envelope(Geometry))
FROM
 "N03-14_140401"
LIMIT 10
```  
  
この結果は以下のように単純な図形になります。  
![spatialite017.png](/image/a0ae8048-3435-1f9c-e489-18e0550541df.png)  
  
では、先ほどと同じ都庁を含む座標をMBRを利用して検索してみましょう。  
  
```sql
SELECT
  *
FROM
 "N03-14_140401"
WHERE
  MBRContains("N03-14_140401".Geometry,GeomFromText('POINT(139.692101 35.689634 )'))
```  
  
検索速度が向上したかわりに以下のように、不要なデータも抽出されてしまっています。  
  
![spatialite018.png](/image/6bd04ff4-7498-d95a-089b-746b6f5a0914.png)  
  
### RTreeインデックスを利用した検索  
通常、DBを扱う場合、インデックスを利用することで検索速度が向上します。  
残念なことに、文字や数値に付与できるインデックスをGeometryに付与することはできません。  
しかしながら、SpatiaLiteはSQLiteのRTreeを利用して、RTreeインデックスをサポートしています。  
RTreeインデックスのアルゴリズムについては下記を参考にしてください。  
  
 **Wonderful R*Tree Spatial Index**  
http://www.gaia-gis.it/gaia-sins/spatialite-cookbook/html/rtree.html  
  
RTreeの概要としては、GeometryをMBRをつかって四角形(rectangle)にし、その四角形の交わりに応じてツリーを作り、検索しやすくしています。  
rectangleを使ったツリーを作成しているのでRTree indexと言われます。  
  
#### RTreeインデックスの作成と利用  
では、RTreeインデックスを実際使用してみます。  
RTreeインデックスを作成するにはCreateSpatialIndex関数を使用します。第一引数にテーブル名、第二引数に列名を指定します。  
  
```sql
SELECT CreateSpatialIndex("N03-14_140401", "Geometry") ;
```  
  
正常に作成できた場合は、1が返り、refresh後にSpatialIndexに新しいテーブルが表示されることが確認できます。  
![spatialite019.png](/image/69b33707-10f5-2918-c61b-71e6d05acafc.png)  
  
![spatialite020.png](/image/affa8866-87f3-5cb6-0908-5fd4a0d4177a.png)  
  
  
RTreeインデックスを作成することにより、以下の４つのテーブルが作成されます。  
  
・idx_N03-14_140401_Geometry  
・idx_N03-14_140401_Geometry_node  
・idx_N03-14_140401_Geometry_parent  
・idx_N03-14_140401_Geometry_rowid  
  
idx_N03-14_140401_Geometry以外は内部で使用するテーブルになっており、ユーザーが直接操作することを期待していません。  
  
idx_N03-14_140401_GeometryはVIRTUAL TABLEになっており、実際には内部テーブルを利用して結果を返します。  
  
では、idx_N03-14_140401_Geometryの内容を確認してみます。  
  
```sql
SELECT * FROM "idx_N03-14_140401_Geometry" LIMIT 10;
```  
  
このように各レコードのMRBが格納されています。  
![spatialite021.png](/image/a463a883-5e38-8fd9-2c3e-80f0f9b3874f.png)  
  
ユーザはこのテーブルを利用することで、大量のデータの中からデータをフィルターをし、結果を求めることができます。  
  
```sql
SELECT 
  * 
FROM "N03-14_140401"
WHERE ROWID IN(
  SELECT
    pkid
  FROM
   "idx_N03-14_140401_Geometry"
  WHERE
    MBRContains(
      BuildMBR(xmin,ymin,xmax,ymax),
      GeomFromText('POINT(139.692101 35.689634 )')
  )
) AND Contains(Geometry, 
  GeomFromText('POINT(139.692101 35.689634 )')
)
```  
  
通常の検索より早く正確なデータが取得できたかと思います。これはデータ数が多くなれば、なるほどその差が大きく現れます。  
  
このようにRTreeインデックスは通常のインデックスと違い、実テーブルにSQLを実行しただけでは適用されずに、作成されたVIRTUAL TABLEを利用してサブクエリ―やJOINでデータをフィルタリングする役割になります。  
  
別テーブルでインデックスを管理していることで、実テーブルが更新された時に不一致がでるかもしれないという疑問をもつでしょう。  
しかし、CreateSpatialIndexを実行した時にトリガーが作成されているので、ユーザは一切手を加えずにインデックスとテーブルの同期がとれます。  
  
このトリガーを確認するには次のようなSQLを実行します。  
  
```sql
select name  from sqlite_master where type = 'trigger' and tbl_name='N03-14_140401';
```  
  
今回は次のようなトリガーが自動で作成されていることが確認できます。  
  
・ggi_N03-14_140401_Geometry  
・ggu_N03-14_140401_Geometry  
・gii_N03-14_140401_Geometry  
・giu_N03-14_140401_Geometry  
・gid_N03-14_140401_Geometry  
  
  
#### RTreeインデックスの削除  
作成したRTreeインデックスを削除するには、DisableSpatialIndexを使用します。  
  
```sql
SELECT DisableSpatialIndex("N03-14_140401", "Geometry") ;
```  
  
refreshを実行するとSpatial Indexが削除されていることが確認できます。  
![spatialite022.png](/image/9c3b455c-792d-669a-3bc3-64fe1f6c1f07.png)  
  
## pythonからの利用  
pythonからSpatiaLiteを使用する場合、SQLiteが適切に利用できる状況なら簡単に利用できます。  
  
Pythonが使っているSqlite3のバージョンは次の手順で調べることができます。  
  
```py
import sqlite3
print (sqlite3.sqlite_version_info)
```  
  
もし、バージョンを満たさない場合は、SQLITEを更新してください。  
Windowsの場合は以下のフォルダのDLLを書き換えるといいでしょう。  
  
C:\Python27\DLLs\sqlite3.dll  
  
以下はPythonでのコードになります。  
  
```py
# !/usr/bin/python
# -*- coding: utf-8 -*-
import sqlite3
import os

# mod_spatialiteのあるフォルダをPATHに加える
os.environ["PATH"] = os.environ["PATH"] + ';C:\\tool\\spatialite\\mod_spatialite-4.2.0-win-x86'
cnn = sqlite3.connect('database/gyouseikuiki.sqlite')

# mod_spatialiteの読み込み
cnn.enable_load_extension(True)
cnn.execute("SELECT load_extension('./mod_spatialite-4.2.0-win-x86/mod_spatialite.dll');")
sql = """
SELECT
  N03_001,
  N03_002,
  N03_003,
  N03_004,
  N03_007, 
  AsGeoJson(Geometry)
FROM
 "N03-14_140401"
WHERE
  MBRContains("N03-14_140401".Geometry,GeomFromText('POINT(139.692101 35.689634 )'))
"""

ret = cnn.execute(sql)
for r in ret:
  print('----------------------------')
  print(r[0].encode('utf_8') )
  print(r[1].encode('utf_8') )
  print(r[2].encode('utf_8') )
  print(r[3].encode('utf_8') )
  print(r[4].encode('utf_8') )
  print(r[5].encode('utf_8') )
```  
  
環境変数をmod_spatialiteのあるパスに通し、enable_load_extensionで拡張DLLの使用を許可してから、mod_spatialiteをロードします。  
  
あとは、通常通りSQLが実行できます。  
  
### enable_load_extensionでエラーになる場合  
enable_load_extensionを使用した時に以下のようなエラーが発生する場合があります。  
  
```
"'sqlite3.Connection' object has no attribute 'enable_load_extension'"
```  
  
原因として、MaxOSやDebianなどのいくつかのOSにプリインストールされているPythonはコンパイル時に、拡張ライブラリの読み込み機能を無効にしているためです。  
  
https://docs.python.org/2/library/sqlite3.html#f1  
  
これに対応するにはPythonをインストールしなおすしかありません。  
インストールする際、Pythonのソースコードのフォルダにある、setup.pyを手で修正する必要があります。  
  
SQLITE_OMIT_LOAD_EXTENSIONを検索して、その行をコメントアウトして、./configure, make, make install を実行します。  
  
もし、make install でエラーが発生する場合は、make install時にエラーがでるなら「make -i altinstall」をためしてみてください。  
  
http://python.g.hatena.ne.jp/nelnal_programing/20101026/1288084718  
  
### Peeweeを利用する方法  
ORMであるPeeweeライブラリでspatialiteを使用する方法については下記を参考にしてください。  
  
http://qiita.com/mima_ita/items/9d4e1d0afac1865acdbb#spatialitesql%E3%81%B8%E3%81%AE%E6%8E%A5%E7%B6%9A%E6%96%B9%E6%B3%95  
  
## phpからの利用  
Windows+PHPの場合、簡単に使うことができません。  
詳細は下記を参照してください。  
  
PHPで地図とかの空間情報をDBに格納しようとしてSpatialiteを使うと苦労する  
http://qiita.com/mima_ita/items/1a90de89f0a194ba843e  
  
  
## その他Tip  
### 測地系の変換をSpatialiteを使って行う  
Transform関数を使うと測地系の変換が行えます。  
以下の例ではJGD2000/平面直角座標系6を世界測地系に変換しています。  
  
```
select AsText(Transform(GeomFromText('POINT(-4408.916645 -108767.765479)', 2448), 4326))
```  
  
## 参考  
 **SpatiaLite 4.2.0  SQL functions reference list**   
http://www.gaia-gis.it/gaia-sins/spatialite-sql-4.2.0.html  
  
 **The SpatiaLite Cookbook**   
http://www.gaia-gis.it/gaia-sins/spatialite-cookbook/index.html  
  
 **A quick tutorial to SpatiaLite - a Spatial extension for SQLite(古いが有用)**   
http://www.gaia-gis.it/gaia-sins/spatialite-tutorial-2.3.1.html  
