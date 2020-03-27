ShapeFileは空間データの位置と形に関する情報と、その属性情報を格納するためのデータ形式です。  
国土数値情報などからダウンロードするとついてくる.shp拡張子のファイルが該当します。  
  
今回はこのShapeFileをPythonで読み取り、Spatialiteに格納するのを目標とします。  
  
  
## ShapeFileの詳細  
ShapeFileは3つのファイルにより構成されます。  
メインファイル、インデックスファイル、属性ファイルの３つです。  
これらのファイル名は拡張子を除き同じものになります。  
  
■メイン・ファイル：counties.shp  
■インデックス・ファイル： counties.shx  
■属性ファイル： counties.dbf   
  
  
メインファイルは空間データが格納されています。  
インデックスファイルは、各空間データへのアクセスを容易にするためのインデックスになります。  
属性ファイルは、属性値が格納されています。  
  
これらの詳細の仕様は下記を参考にしてください。  
  
 **シェープファイルの技術情報**   
http://www.esrij.com/cgi-bin/wp/wp-content/uploads/documents/shapefile_j.pdf  
  
  
## PythonでShapeFileを操作する  
PythonでShapeFileを操作するには下記のライブラリを用いると良いでしょう。  
  
https://github.com/GeospatialPython/pyshp  
  
 **インストール方法**  
shapefile.py を任意のフォルダに配置してimportする。  
  
このライブラリはpython2.4-3.x系で利用できます。  
  
### 国土数値情報の操作例  
この例では、国土数値情報の鉄道路線情報のN02-05-g_RailroadSection.shpを操作してみます。  
  
 **国土数値情報　鉄道データ**   
http://nlftp.mlit.go.jp/ksj/gml/datalist/KsjTmplt-N02-v2_2.html  
  
  
```py
# -*- coding: utf-8 -*-
import os
import sys
sys.path.append(os.path.dirname(os.path.abspath(__file__)) + '/pyshp')
import shapefile

sf = shapefile.Reader('original_data\\N02-05_GML\\N02-05\\N02-05-g_RailroadSection.shp')
shapeRecs = sf.iterShapeRecords()
for sr in shapeRecs:
  #属性値が入っている
  print ('attribute:' , sr.record)

  #型のタイプ
  #NULL = 0
  #POINT = 1
  #POLYLINE = 3
  #POLYGON = 5
  #MULTIPOINT = 8
  #POINTZ = 11
  #POLYLINEZ = 13
  #POLYGONZ = 15
  #MULTIPOINTZ = 18
  #POINTM = 21
  #POLYLINEM = 23
  #POLYGONM = 25
  #MULTIPOINTM = 28
  #MULTIPATCH = 31
  print ('shapeType:' ,sr.shape.shapeType)

  #座標ポイントのリスト
  print ('points:', sr.shape.points)

  #MultiLingやMultiPolygonの際に、pointsをどこで分割するか
  print ('parts:' ,sr.shape.parts)
```  
  
iterShapeRecords()は、shpファイルを頭から解析していきます。  
この際、メモリに展開するのは１データのみなので、大データの処理に適します。  
  
しかしながら、iterShapeRecordはレコードヘッダに記録されているコンテンツ長の次のバイトに次のレコードがあることが前提になっています。  
多くの場合は、この前提で解析できるのですが、この前提が正しくない場合があります。  
たとえば、以下のA31-12_17_GML.shpを実行するとエラーになります。  
  
 **国土数値情報　浸水想定区域データの石川県**   
http://nlftp.mlit.go.jp/ksj/gml/datalist/KsjTmplt-A31.html  
  
これは、レコードの間にゴミがはいっているため、shpファイルだけでは解析できず、インデックスファイルを利用する必要があります。  
  
この場合は、以下のように、iterShapeRecordsを使用せずに実装することが可能です。  
  
```py
# -*- coding: utf-8 -*-
import os
import sys
sys.path.append(os.path.dirname(os.path.abspath(__file__)) + '/pyshp')
import shapefile

sf = shapefile.Reader('original_data\\A31-12\\output\\A31-12_17_GML\\old\\A31-12_17.shp')
try:
    # 国土数値情報のshpファイルが不正で、shpファイル中のコンテンツ長と実際の長さのつじつまが合っていない。
    # しかたないのでshxファイル経由で各レコードを取得する
    i = 0
    while True:
        shape = sf.shape(i)
        rec = sf.record(i)
        #属性値が入っている
        print ('attribute:' , rec)

        #型のタイプ
        #NULL = 0
        #POINT = 1
        #POLYLINE = 3
        #POLYGON = 5
        #MULTIPOINT = 8
        #POINTZ = 11
        #POLYLINEZ = 13
        #POLYGONZ = 15
        #MULTIPOINTZ = 18
        #POINTM = 21
        #POLYLINEM = 23
        #POLYGONM = 25
        #MULTIPOINTM = 28
        #MULTIPATCH = 31
        print ('shapeType:' ,shape.shapeType)

        #座標ポイントのリスト
        print ('points:', shape.points)

        #MultiLingやMultiPolygonの際に、pointsをどこで分割するか
        print ('parts:' , shape.parts)

        i += 1
except IndexError:
    pass
```  
  
  
これを利用すれば、shapeファイルをPythonで解析してspatialiteにインポートするという処理が行えます。  
  
以下のプログラムでは国土数値情報の土砂災害危険箇所データ、浸水想定区域データ、竜巻等の突風データをShapeファイルからspatialiteに格納しています。  
https://github.com/mima3/kokudo/blob/master/kokudo_db.py  
  
 **Demo**   
http://needtec.sakura.ne.jp/kokudo/  
  
SPATIALITEの使い方については下記の記事を参考にしてください。  
http://qiita.com/mima_ita/items/64f6c2b8bb47c4b5b391  
