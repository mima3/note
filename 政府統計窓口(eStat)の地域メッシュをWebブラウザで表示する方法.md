## 概要  
地域メッシュ統計とは，緯度・経度に基づき地域を隙間なく網の目（Mesh）の区域に分けて，統計データをそれぞれの区域に編成したものです。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/bb9f0191-60eb-8b40-3725-309897a1d4d5.png  
※色が濃いほど人口が多い。  
  
デモ：  
http://needtec.sakura.ne.jp/estat/population  
http://needtec.sakura.ne.jp/tokuraku/passenger.html  
  
  
このデータの元になっている情報は政府統計窓口(estat)から取得しています。  
以前はPythonでmatplotlib で描画していましたが、ブラウザで表示するようにしました。  
  
 **政府統計の総合窓口(e-Stat)のAPIを使ってみよう**   
http://qiita.com/mima_ita/items/44f358dc1bc4000d365d  
  
  
なお、地域メッシュの詳しい求め方は以下を参考にしてください。  
http://www.stat.go.jp/data/mesh/pdf/gaiyo1.pdf  
  
## データ描画までの流れ  
描画のたびに、政府統計窓口に問い合わせるっていうのは、遅すぎるのでデータ描画までの手順を以下のように分けます。  
  
1.ある統計に紐づく地域メッシュ情報を政府統計窓口より、取得する  
2.spatialiteに記録する  
3.Webサーバーでリクエストに応じてgeojson形式でメッシュ情報を返すようにする。  
4.画面側は必要な領域だけWebサーバーに要求して取得したgeojsonを元に色を塗る。  
  
これらの手順をサーバーサイドはPython、クライアントサイドはJavaScriptで実施します。  
  
実際のコードは下記から取得できます。  
https://github.com/mima3/estat  
  
  
spatialiteについての説明は下記を参考にしてください。  
 **地図とかの空間情報をSQLiteに格納するSpatiaLiteを使用してみる**   
http://qiita.com/mima_ita/items/64f6c2b8bb47c4b5b391  
  
## 地域メッシュを格納するデータベースの作成  
ここで紹介する統計情報を格納するデータベースの実装は以下のコードになります。  
https://github.com/mima3/estat/blob/master/estat_db.py  
  
### テーブル構成  
estatの地域メッシュを格納するためのテーブルは以下のようになります。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/43b7de44-900f-6ce6-0246-3499e619dd1d.png  
  
  
 **Table名：Stat**   
統計に対する情報を格納するテーブル  
  
|列名|型|説明|  
|:--|:--|:--|  
|stat_id|TextField|主キー。統計情報のID|  
|stat_name|TextField|政府統計名|  
|stat_name_code|TextField|政府統計名コード|  
|gov_org|TextField|作成機関名|  
|gov_org_code|TextField|作成機関名コード|  
|survey_date|TextField|調査日|  
|title|TextField|統計表の表題|  
  
  
 **Table名：StatValue**   
各統計の値を格納する  
  
|列名|型|説明|  
|:--|:--|:--|  
|id|PrimaryKeyField|自動インクリメントのID|  
|stat_id|TextField|統計情報のID|  
|value|TextField|統計の値|  
  
  
 **Table名:StatValueAttr**   
各統計の値に紐づく属性値を格納する  
  
|列名|型|説明|  
|:--|:--|:--|  
|id|PrimaryKeyField|自動インクリメントのID||  
|stat_id|TextField|統計情報のID|  
|stat_value_id|INT|StatValueのidの外部キー|  
|attr_name|TextField|属性名|  
|attr_value|TextField|属性値|  
  
  
 **Table名:MapArea**   
属性areaにおけるPolygonを格納する  
  
|列名|型|説明|  
|:--|:--|:--|  
|id|PrimaryKeyField|自動インクリメントのID|  
|stat_id|TextField|統計情報のID|  
|stat_val_attr_id|INT|StatValueAttrのID|  
|geometry|Polygon|ジオメトリ情報|  
  
### Pythonでテーブルの作成方法  
pythonでpeeweeを用いて作成する場合の実際のコードは以下のようになります。  
  
```py:estat_db.py
# -*- coding: utf-8 -*-
import os
from peewee import *
from playhouse.sqlite_ext import SqliteExtDatabase
from estat_go_jp import *
import jpgrid


database_proxy = Proxy()  # Create a proxy for our db.

SRID = 4326


class PolygonField(Field):
    db_field = 'polygon'


class PointField(Field):
    db_field = 'point'


class LineStringField(Field):
    db_field = 'linestring'


class MultiPolygonField(Field):
    db_field = 'multipolygon'


class MultiPointField(Field):
    db_field = 'multipoint'


class MultiLineStringField(Field):
    db_field = 'multilinestring'


class Stat(Model):
    """
    統計情報を格納するモデル
    """
    stat_id = TextField(primary_key=True)
    stat_name = TextField()
    stat_name_code = TextField()
    gov_org = TextField()
    gov_org_code = TextField()
    #statistics_name = TextField()
    #cycle = TextField()
    survey_date = TextField()
    #open_date = TextField()
    #small_area = TextField()
    title = TextField()

    class Meta:
        database = database_proxy


class StatValue(Model):
    """
    統計の値を格納するモデル
    """
    id = PrimaryKeyField()
    stat_id = TextField(index=True, unique=False)
    value = TextField()

    class Meta:
        database = database_proxy


class StatValueAttr(Model):
    """
    統計の属性値を格納するモデル
    """
    id = PrimaryKeyField()
    stat_id = TextField()
    stat_value = ForeignKeyField(
        db_column='stat_value_id',
        rel_model=StatValue,
        to_field='id'
    )
    attr_name = TextField()
    attr_value = TextField()

    class Meta:
        database = database_proxy
        indexes = (
            (('stat_id', 'stat_value'), False),
            (('stat_id', 'stat_value', 'attr_name'), False),
        )


class MapArea(Model):
    """
    統計の属性値 areaにおけるPolygonを格納するモデル
    """
    id = PrimaryKeyField()
    stat_id = TextField()
    stat_val_attr = ForeignKeyField(
        db_column='stat_val_attr_id',
        rel_model=StatValueAttr,
        to_field='id'
    )
    geometry = PolygonField()

    class Meta:
        database = database_proxy
        indexes = (
            (('stat_id', 'stat_val_attr'), True),
        )


class idx_MapArea_Geometry(Model):
    pkid = PrimaryKeyField()
    xmin = FloatField()
    xmax = FloatField()
    ymin = FloatField()
    ymax = FloatField()

    class Meta:
        database = database_proxy


def connect(path, spatialite_path, evn_sep=';'):
    """
    データベースへの接続
    @param path sqliteのパス
    @param spatialite_path mod_spatialiteへのパス
    @param env_sep 環境変数PATHの接続文字 WINDOWSは; LINUXは:
    """
    os.environ["PATH"] = os.environ["PATH"] + evn_sep + os.path.dirname(spatialite_path)
    db = SqliteExtDatabase(path)
    database_proxy.initialize(db)
    db.field_overrides = {
        'polygon': 'POLYGON',
        'point': 'POINT',
        'linestring': 'LINESTRING',
        'multipolygon': 'MULTIPOLYGON',
        'multipoint': 'MULTIPOINT',
        'multilinestring': 'MULTILINESTRING',
    }
    db.load_extension(os.path.basename(spatialite_path))


def setup(path, spatialite_path, evn_sep=';'):
    """
    データベースの作成
    @param path sqliteのパス
    @param spatialite_path mod_spatialiteへのパス
    @param env_sep 環境変数PATHの接続文字 WINDOWSは; LINUXは:
    """
    connect(path, spatialite_path, evn_sep)

    database_proxy.create_tables([Stat, StatValue, StatValueAttr], True)
    database_proxy.get_conn().execute('SELECT InitSpatialMetaData()')

    # Geometryのテーブルは直接実装する必要がある.
    database_proxy.get_conn().execute("""
        CREATE TABLE IF NOT EXISTS "MapArea" (
          "id" INTEGER PRIMARY KEY AUTOINCREMENT,
          "stat_id" TEXT,
          "stat_val_attr_id" INTEGER ,
          FOREIGN KEY(stat_val_attr_id) REFERENCES StatValueAttr(id));
    """)

    database_proxy.get_conn().execute("""
        CREATE INDEX IF NOT EXISTS "ix_MapArea_stat_id" ON MapArea(stat_id);
    """)

    database_proxy.get_conn().execute("""
        CREATE INDEX IF NOT EXISTS "ix_MapArea_stat_id_stat_val_attr_id" ON MapArea(stat_id, stat_val_attr_id);
    """)

    database_proxy.get_conn().execute("""
        Select AddGeometryColumn ("MapArea", "Geometry", ?, "POLYGON", 2);
    """, (SRID,))

    database_proxy.get_conn().execute("""
        SELECT CreateSpatialIndex("MapArea", "geometry")
    """)
```  
  
setup関数でテーブルの作成を行っています。  
ジオメトリを含まない場合、peeweeでは以下のコードでテーブルを生成します。  
  
```py
database_proxy.create_tables([Stat, StatValue, StatValueAttr], True)
```  
  
しかし、ジオメトリーの列を含む場合、以下のように、自分で実装する必要があります  
  
```py
    # Geometryのテーブルは直接実装する必要がある.
    database_proxy.get_conn().execute("""
        CREATE TABLE IF NOT EXISTS "MapArea" (
          "id" INTEGER PRIMARY KEY AUTOINCREMENT,
          "stat_id" TEXT,
          "stat_val_attr_id" INTEGER ,
          FOREIGN KEY(stat_val_attr_id) REFERENCES StatValueAttr(id));
    """)

    database_proxy.get_conn().execute("""
        CREATE INDEX IF NOT EXISTS "ix_MapArea_stat_id" ON MapArea(stat_id);
    """)

    database_proxy.get_conn().execute("""
        CREATE INDEX IF NOT EXISTS "ix_MapArea_stat_id_stat_val_attr_id" ON MapArea(stat_id, stat_val_attr_id);
    """)

    database_proxy.get_conn().execute("""
        Select AddGeometryColumn ("MapArea", "Geometry", ?, "POLYGON", 2);
    """, (SRID,))

    database_proxy.get_conn().execute("""
        SELECT CreateSpatialIndex("MapArea", "geometry")
    """)
```  
  
このコードではジオメトリー以外の列を含んだテーブルを作成したあとに、「AddGeometryColumn」でジオメトリーの列を追加し、それに対し「CreateSpatialIndex」でインデックスを作成します。  
  
ジオメトリーに対して作られたインデックスは仮想テーブルですので、Peeweeから使用することができません。インデックスを利用する際は、自分でSQLを記述する必要があります。  
  
### Pythonで統計情報をデータベースにインポートする方法  
eStatの地域メッシュでは、一つの統計について、第一次メッシュごとに分割されて統計IDをもっています。  
たとえば、「平成２２年国勢調査－世界測地系(1KMメッシュ)20101001」の統計情報では、以下のような統計IDが取得できます。  
  
・T000608M3622  
・T000608M3623  
・T000608M3624  
・T000608M3653  
略  
  
このIDは「T000608」と、第一次メッシュコードから作成されています。  
  
つまり、地域メッシュの統計情報を取得するには次のような手順が必要になります。  
  
1.eStatのgetStatsList　APIで該当の統計をすべて取得する。  
2.eStatのgetStatsData　APIで統計ID毎に値を取得する。  
3.DBに格納する際に、メッシュコードから経度緯度の範囲をもとめて、Polygonとして格納する。  
4.全ての統計IDに対して2~3を繰り返す  
  
  
  
#### getStatesList APIによる該当の統計を全て取得する  
「平成２２年国勢調査－世界測地系(1KMメッシュ)20101001」に関係する統計IDをすべて取得するためのgetStatesList APIを実行するための関数は以下のように実装します。  
  
参照元：  
https://github.com/mima3/estat/blob/master/estat_go_jp.py  
  
```py:estat_go_jp.py
def get_stats_list(api_key, search_kind, key_word):
    """
    統計情報の検索
    @param api_key APIキー
    @param search_kind 統計の種類
    @param key_word 検索キー
    """
    key_word = urllib.quote(key_word.encode('utf-8'))
    url = ('http://api.e-stat.go.jp/rest/1.0/app/getStatsList?appId=%s&lang=J&searchKind=%s&searchWord=%s' % (api_key, search_kind, key_word))
    req = urllib2.Request(url)
    opener = urllib2.build_opener()
    conn = opener.open(req)
    cont = conn.read()
    parser = etree.XMLParser(recover=True)
    root = etree.fromstring(cont, parser)
    ret = []
    data_list = root.find('DATALIST_INF')
    list_infs = data_list.xpath('.//LIST_INF')
    for list_inf in list_infs:
        item = {
            'id': trim_data(list_inf.get('id'))
        }
        stat_name = list_inf.find('STAT_NAME')
        if stat_name is not None:
            item['stat_name'] = trim_data(stat_name.text)
            item['stat_name_code'] = trim_data(stat_name.get('code'))

        gov_org = list_inf.find('GOV_ORG')
        if gov_org is not None:
            item['gov_org'] = trim_data(gov_org.text)
            item['gov_org_code'] = trim_data(gov_org.get('code'))

        statistics_name = list_inf.find('STATISTICS_NAME')
        if statistics_name is not None:
            item['statistics_name'] = trim_data(statistics_name.text)

        title = list_inf.find('TITLE')
        if title is not None:
            item['title'] = trim_data(title.text)

        cycle = list_inf.find('CYCLE')
        if cycle is not None:
            item['cycle'] = cycle.text

        survey_date = list_inf.find('SURVEY_DATE')
        if survey_date is not None:
            item['survey_date'] = trim_data(survey_date.text)

        open_date = list_inf.find('OPEN_DATE')
        if open_date is not None:
            item['open_date'] = trim_data(open_date.text)

        small_area = list_inf.find('SMALL_AREA')
        if small_area is not None:
            item['small_area'] = trim_data(small_area.text)

        ret.append(item)
    return ret

```  
  
基本的にurllib2で取得した内容をlxmlで解析してディレクトリに格納しているだけです。  
  
#### getStatsData APIで統計ID毎に値を取得する。  
各統計ID毎の値を取得するには以下のget_stats_id_value()を実行します。  
  
参照元：  
https://github.com/mima3/estat/blob/master/estat_go_jp.py  
  
```py:estat_go_jp.py
def get_meta_data(api_key, stats_data_id):
    """
    メタ情報取得
    """
    url = ('http://api.e-stat.go.jp/rest/1.0/app/getMetaInfo?appId=%s&lang=J&statsDataId=%s' % (api_key, stats_data_id))
    req = urllib2.Request(url)
    opener = urllib2.build_opener()
    conn = opener.open(req)
    cont = conn.read()
    parser = etree.XMLParser(recover=True)
    root = etree.fromstring(cont, parser)
    class_object_tags = root.xpath('//METADATA_INF/CLASS_INF/CLASS_OBJ')
    class_object = {}

    for class_object_tag in class_object_tags:
        class_object_id = class_object_tag.get('id')
        class_object_name = class_object_tag.get('name')
        class_object_item = {
            'id': trim_data(class_object_id),
            'name': trim_data(class_object_name),
            'objects': {}
        }
        class_tags = class_object_tag.xpath('.//CLASS')
        for class_tag in class_tags:
            class_item = {
                'code': trim_data(class_tag.get('code')),
                'name': trim_data(class_tag.get('name')),
                'level': trim_data(class_tag.get('level')),
                'unit': trim_data(class_tag.get('unit'))
            }
            class_object_item['objects'][class_item['code']] = class_item
        class_object[class_object_id] = class_object_item
    return class_object

def _get_stats_id_value(api_key, stats_data_id, class_object, start_position, filter_str):
    """
    統計情報の取得
    """
    url = ('http://api.e-stat.go.jp/rest/1.0/app/getStatsData?limit=10000&appId=%s&lang=J&statsDataId=%s&metaGetFlg=N&cntGetFlg=N%s' % (api_key, stats_data_id, filter_str))
    if start_position > 0:
        url = url + ('&startPosition=%d' % start_position)
    req = urllib2.Request(url)
    opener = urllib2.build_opener()
    conn = opener.open(req)
    cont = conn.read()
    parser = etree.XMLParser(recover=True)
    root = etree.fromstring(cont, parser)
    ret = []
    row = {}
    datas = {}
    value_tags = root.xpath('//STATISTICAL_DATA/DATA_INF/VALUE')
    for value_tag in value_tags:
        row = {}
        for key in class_object:
            val = value_tag.get(key)
            if val in class_object[key]['objects']:
                text = class_object[key]['objects'][val]['name']
                row[key] = trim_data(text.encode('utf-8'))
            else:
                row[key] = val.encode('utf-8')
        row['value'] = trim_data(value_tag.text)
        ret.append(row)
    return ret


def get_stats_id_value(api_key, stats_data_id, filter):
    """
    統計取得
    @param api_key APIキー
    @param stats_data_id 統計表ID
    @param filter_str フィルター文字
    """
    filter_str = ''
    for key in filter:
        filter_str += ('&%s=%s' % (key, urllib.quote(filter[key].encode('utf-8'))))
    class_object = get_meta_data(api_key, stats_data_id)
    return _get_stats_id_value(api_key, stats_data_id, class_object, 1, filter_str), class_object

```  
  
get_stats_id_valueはフィルターを掛けて検索できますが、今回はフィルターなしで全て取得するので、そこは考慮しないでください。  
  
手順としては、get_meta_dataにより、統計情報のメタ情報を取得します。これにより、該当の統計がどのような属性を有しているか調べます。  
  
その後、_get_stats_id_value()を用いて、統計値と、その属性の値を取得します。  
  
  
#### DBに格納する際に、メッシュコードから経度緯度の範囲をもとめて、Polygonとして格納する  
eStatから取得するデータには残念なことに経度緯度の情報は存在しません。  
そこで、メッシュコードから経度、緯度を取得する必要があります。  
Pythonの場合、以下のライブラリを使用すると楽でしょう。  
  
 **python-geohash**   
https://code.google.com/p/python-geohash/  
  
このライブラリは以下のように、easy_install等でインストール可能です。  
  
```
easy_install python-geohash
```  
  
使い方はメッシュコードを指定するだけ、該当のメッシュコードの範囲が、経度と緯度で表示されます。  
  
```py
import jpgrid
jpgrid.bbox('305463') #メッシュコードを指定する →　{'s': 154.375, 'e': 20.583333333333332, 'w': 20.5, 'n': 154.5}
```  
  
これらを利用して、データベースに地域メッシュの統計情報を格納するコードは次のようになります。  
  
参照元：  
https://github.com/mima3/estat/blob/master/estat_db.py  
  
```py:estat_db.py
def import_stat(api_key, stat_id):
    """
    統計情報のインポート
    @param api_key e-statのAPIKEY
    @param stat_id 統計ID
    """
    with database_proxy.transaction():
        MapArea.delete().filter(MapArea.stat_id == stat_id).execute()
        StatValueAttr.delete().filter(StatValueAttr.stat_id == stat_id).execute()
        StatValue.delete().filter(StatValue.stat_id == stat_id).execute()
        Stat.delete().filter(Stat.stat_id == stat_id).execute()

        tableinf = get_table_inf(api_key, stat_id)
        stat_row = Stat.create(
            stat_id=stat_id,
            stat_name=tableinf['stat_name'],
            stat_name_code=tableinf['stat_name_code'],
            title=tableinf['title'],
            gov_org=tableinf['gov_org'],
            gov_org_code=tableinf['gov_org_code'],
            survey_date=tableinf['survey_date']
        )
        values, class_object = get_stats_id_value(api_key, stat_id, {})
        for vdata in values:
            if not 'value' in vdata:
                continue
            value_row = StatValue.create(
                stat_id=stat_id,
                value=vdata['value']
            )
            for k, v in vdata.items():
                stat_val_attr = StatValueAttr.create(
                    stat_id=stat_id,
                    stat_value=value_row,
                    attr_name=k,
                    attr_value=v
                )
                if k == 'area':
                    # メッシュコード
                    meshbox = jpgrid.bbox(v)
                    database_proxy.get_conn().execute(
                        'INSERT INTO MapArea(stat_id, stat_val_attr_id, geometry) VALUES(?,?,BuildMBR(?,?,?,?,?))',
                        (stat_id, stat_val_attr.id, meshbox['s'], meshbox['e'], meshbox['n'], meshbox['w'], SRID)
                    )
        database_proxy.commit()
```  
  
属性名がareaの場合、地域メッシュとして、MapAreaにジオメトリ情報を格納します。  
R-Indexを用いている場合、PeeweeのORMで動作させるのが困難なので、ここは直接SQL文を記述しています。  
  
#### インポート用のスクリプト  
以上を利用したインポート用のスクリプトが次の通りです。  
  
https://github.com/mima3/estat/blob/master/import_estat.py  
  
使い方はAPI_KEY、統計のタイトル、mod_spatialite.dllへのパス、DBへのパスを指定して実行します。  
  
```
python import_estat.py API_KEY 2 平成２２年国勢調査－世界測地系(1KMメッシュ)20101001  C:\tool\spatialite\mod_spatialite-4.2.0-win-x86\mod_spatialite.dll estat.sqlite
```  
  
このスクリプトはWindows用にcp932で文字コードをとりあつかっているので、お使いのターミナルに合わせて修正してください。  
  
  
## WebサーバーでGeoJSONとして地域メッシュの情報を返す  
次にWebサーバーで指定の範囲の地域メッシュをGeoJSONとして返す方法について説明します。  
ここでは、bottleを用いていますが、この辺りは好きなWebフレームワークで構わないと思いますし、Webフレームワークを使うまでもないかもしれません。  
  
 **Bottle: Python Web Framework**   
http://bottlepy.org/docs/dev/index.html  
  
まず、範囲を指定して、DB中から該当の地域メッシュを取得するようなコードを記述します。  
  
```py:estat_db.py
def get_mesh_stat(stat_id_start_str, attr_value, xmin, ymin, xmax, ymax):
    """
    地域メッシュの統計情報を取得する
    @param stat_id_start_str 統計IDの開始文字 この文字から始まるIDをすべて取得する.
    @param attr_value cat01において絞り込む値
    @param xmin 取得範囲
    @param ymin 取得範囲
    @param xmax 取得範囲
    @param ymax 取得範囲
    """
    rows = database_proxy.get_conn().execute("""
        SELECT
          statValue.value,
          AsGeoJson(MapArea.Geometry)
        FROM
          MapArea
          inner join idx_MapArea_Geometry ON pkid = MapArea.id AND xmin > ? AND ymin > ? AND xmax < ? AND ymax < ?
          inner join statValueAttr ON MapArea.stat_val_attr_id = statValueAttr.id
          inner join statValueAttr AS b ON b.stat_value_id = statValueAttr.stat_value_id AND b.attr_value = ?
          inner join statValue ON statValue.id = b.stat_value_id
        WHERE
          MapArea.stat_id like ?;
    """, (xmin, ymin, xmax, ymax, attr_value, stat_id_start_str + '%'))
    ret = []
    for r in rows:
        ret.append({
            'value': r[0],
            'geometory': r[1]
        })
    return ret
```  
  
GeoJSONとして扱うために、AsGeoJson(MapArea.Geometry)でGeometryをGeoJSONに変換しています。  
これを以下のように、結合して、一つのGeoJSONとしてクライアントに返します。  
  
参照元：  
https://github.com/mima3/estat/blob/master/application.py  
  
```py:application.py
@app.get('/json/get_population')
def getPopulation():
    stat_id = request.query.stat_id
    swlat = request.query.swlat
    swlng = request.query.swlng
    nelat = request.query.nelat
    nelng = request.query.nelng
    attrval = request.query.attr_value

    ret = estat_db.get_mesh_stat(stat_id, attrval, swlng, swlat, nelng, nelat)
    res = {'type': 'FeatureCollection', 'features': []}
    for r in ret:
        item = {
            'type': 'Feature',
            'geometry': json.loads(r['geometory']),
            'properties': {'value': r['value']}
        }
        res['features'].append(item)
    response.content_type = 'application/json;charset=utf-8'
    return json.dumps(res)
```  
  
これにより、次のようなリクエストが処理できるようになります。  
  
```
http://needtec.sakura.ne.jp/estat/json/get_population?stat_id=T000608&attr_value=%E4%BA%BA%E5%8F%A3%E7%B7%8F%E6%95%B0&swlat=35.503426100823496&swlng=139.53192492382811&nelat=35.83811583873688&nelng=140.08124133007811
```  
  
レスポンス：  
  
```json
{"type": "FeatureCollection", "features": [{"geometry": {"type": "Polygon", "coordinates": [[[139.5374999999999, 35.6], [139.5499999999999, 35.6], [139.5499999999999, 35.60833333333333], [139.5374999999999, 35.60833333333333], [139.5374999999999, 35.6]]]},略 
```  
  
## GoogleMapを用いた地域メッシュの描画の例  
ここではGoogleMapを用いた地域メッシュの描画例を記述します。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/ec026d58-dc6e-2f97-0302-4ed8cada659f.png  
  
デモ：  
http://needtec.sakura.ne.jp/estat/population  
  
  
GoogleMapではaddGeoJSONを用いることで、任意のGeoJSONをGoogleMAP上に描画できます。  
  
```js:population.js
          features  = map.data.addGeoJson(result);
          var max = 0;
          for (var i = 0; i < features.length; i++) {
            if (max < features[i].getProperty('value')) {
              max = features[i].getProperty('value');
            }
          }
          map.data.setStyle(styleFeature(max));
```  
  
この際、setStyleで、GeoJSONのスタイルを指定できます。この例ではプロパティvalueに応じて色の濃さを変更しています。  
  
```js:population.js
      var styleFeature = function(max) {
        var colorScale = d3.scale.linear().domain([0, max]).range(["#CCFFCC", "red"]);
        return function(feature) {
          return {
            strokeWeight : 1,
            fillColor: colorScale(+feature.getProperty('value')),
            fillOpacity: 0.5
          };
        };
      }
```  
  
参考：  
 **Google Map上にGeoJSONデータを表示する**   
http://shimz.me/blog/google-map-api/3445  
  
あまり、大きな範囲を一度に描画すると重いので、拡大機能を無効にしています。  
  
## D3.jsを用いた地域メッシュの描画の例  
ここではD3.jsを用いてSVGによる地域メッシュの描画例を記述します。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/243b6691-f958-52dd-8b15-ddf93c91e8a4.png  
  
デモ：  
http://needtec.sakura.ne.jp/tokuraku/passenger.html  
  
```js:passenger.js
$('#selMesh').change(function() {
  svgMeshGrp
     .attr('class', 'tracts')
     .selectAll('rect')
     .data([])
     .exit()
     .remove();

  var sel = $('#selMesh').val();
  if (!sel) {
    return;
  }

  function drawMesh(json) {
    console.log(json);
    var max = 0;
    for (var i = 0; i < json.features.length; ++i) {
      var v = parseInt(json.features[i].properties.value);
      if (max < v) {
        max = v;
      }
    }
    console.log(max);
    var colorScale = d3.scale.linear().domain([0, max]).range([0.0, 0.8]);
    svgMeshGrp
       .attr('class', 'tracts')
       .selectAll('rect')
       .data(json.features)
       .enter()
       .append('rect')
       .attr('x', function(d, i) {
         var extX = d3.extent(d.geometry.coordinates[0], function(d) { return d[0];});
         var extY = d3.extent(d.geometry.coordinates[0], function(d) { return d[1];});
         var pt = projection([extX[0], extY[0]]);
         return pt[0];
       })
       .attr('y', function(d) {
         var extX = d3.extent(d.geometry.coordinates[0], function(d) { return d[0];});
         var extY = d3.extent(d.geometry.coordinates[0], function(d) { return d[1];});
         var pt = projection([extX[0], extY[0]]);
         return pt[1];
       })
       .attr('width', function(d) {
         var extX = d3.extent(d.geometry.coordinates[0], function(d) { return d[0];});
         var extY = d3.extent(d.geometry.coordinates[0], function(d) { return d[1];});
         var ptMin = projection([extX[0], extY[0]]);
         var ptMax = projection([extX[1], extY[1]]);
         return Math.abs(ptMax[0] - ptMin[0]);
       })
       .attr('height', function(d) {
         var extX = d3.extent(d.geometry.coordinates[0], function(d) { return d[0];});
         var extY = d3.extent(d.geometry.coordinates[0], function(d) { return d[1];});
         var ptMin = projection([extX[0], extY[0]]);
         var ptMax = projection([extX[1], extY[1]]);
         return Math.abs(ptMax[1] - ptMin[1]);
       })
       .attr('fill-opacity', function(d) {
         console.log('color' , d.properties.value, colorScale(d.properties.value));
         return colorScale(d.properties.value);
       })
       .attr('fill' , '#00f');
  }

  if (stat_dict[sel].json) {
    drawMesh(stat_dict[sel].json);
  } else {
    $.blockUI({ message: '<img src="/railway_location/img/loading.gif" />' });
    var api = encodeURI(util.format('/estat/json/get_population?swlat=%d&swlng=%d&nelat=%d&nelng=%d&stat_id=%s&attr_value=%s',
      swlat, swlng, nelat, nelng, stat_dict[sel].id, stat_dict[sel].attrval
    ));
    d3.json(api, function(json) {
      drawMesh(json);
      stat_dict[sel].json = json;
      $.unblockUI();
    });
  }
}).keyup(function() {
  $(this).blur().focus();
});
```  
  
ここでの注意点は、GeoJSONを使用して、pathとして描画せずに、rectで描画している点です。  
svgにpathを1000個追加すると重くなりますが、rectであればそれよりも軽いです。  
地域メッシュは必ず四角形なのでrectで描画した方がいいでしょう。  
  
## まとめ  
政府総合の統計窓口eStatで地域メッシュを記述する方法を説明しました。  
SpatialiteなどによるGeometry情報を格納できるDBに一旦格納することで、処理速度があがることが実証できたかと思います。  
