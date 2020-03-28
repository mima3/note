## 目的  
国土数値情報の行政区域の東京都と神奈川県の情報を編集した後にブラウザに表示してみます。  
  
![map012.png](/image/5c6713b3-945f-3482-5a65-2f2bb7f8d86c.png)  
  
 **Demo**   
http://needtec.sakura.ne.jp/dc_example/simpleMap.html  
  
## データの入手  
国土数値情報ダウンロードサービス から 東京都と神奈川県の情報を取得します。  
http://nlftp.mlit.go.jp/ksj/gml/datalist/KsjTmplt-N03.html  
  
以下のフォルダが取得できます。  
  
N03-140401_13_GML : 東京都  
N03-140401_14_GML : 神奈川県  
  
このフォルダ中のN03-14_13_140401.shpとN03-14_14_140401.shpを使用します。  
  
## QGISで地図情報を編集する。  
### QGISのインストール  
QGISを下記からダウンロードしてインストールします。  
  
http://qgis.org/ja/site/  
  
Windows,Mac,Linuxなどの複数のOSで動作します。  
  
### shpファイルを開く  
1.QGIS Desktopを開きます  
![map001.png](/image/dc5d6a5a-608d-1c98-f9b1-18bbc6101e52.png)  
  
2.画面が立ち上がるまでゆっくり待ちます(結構遅い）  
  
![map002.png](/image/840e0e72-4113-4dab-385d-337aa3b45bec.png)  
  
3.「レイヤ」→「レイヤの追加」→「ベクタレイヤの追加」を選択します  
![map003.png](/image/2877f84d-892d-04ad-597a-eff15455ad94.png)  
  
4.神奈川県の情報であるN03-14_14_140401.shpを選択します。エンコーディングは「System」とします。  
![map012.png](/image/a58b8598-be84-36a0-32ed-991dd5525893.png)  
  
5.神奈川県の地図が表示されます  
![map012.png](/image/f834b129-b3c1-d166-2787-b3f18ffc6488.png)  
  
  
### 属性テーブルの文字を確認する  
国土数値情報の行政区域には県や市の名前が属性として設定されています。  
この値が正しく表示されているか確認します。  
  
1.メニューのアイコンから属性テーブルを開きます  
![map006.png](/image/0e3528a6-52f1-4708-cc02-300fd266022e.png)  
  
2.Windowsの場合、以下のように文字化けしています。  
![map011.png](/image/b3119408-0c18-32dd-f7f5-404c9a54a3c5.png)  
  
  
### 東京都と神奈川県を合わせる  
東京都と神奈川県を同じレイヤに表示する。  
1.N03-14_13_140401.shpを同様の手順で表示します。  
  
![map012.png](/image/81ad6645-68b7-c32e-954e-e2af07a52b74.png)  
  
2.メニューから「領域またはシングルクリックによる地物選択」を選択します。  
![map012.png](/image/c16ffd14-8582-2762-140a-5c09d8ff46c7.png)  
  
3.東京都の必要な個所を選択します。この際、マウスホイールにより地図の縮小率を変更できます。  
  
![map012.png](/image/1f24cbcf-a32c-19c9-d45b-45c64b52c795.png)  
  
選択した範囲は以下のように色が変更されます。  
![map012.png](/image/238c2906-3193-0711-51d5-237b9ccd4319.png)  
  
4.「CTRL+C」または「編集」→「地物のコピー」で指定範囲をコピーします。  
![map012.png](/image/481c8a3b-96a9-56d4-adc7-ae22c23defdf.png)  
  
5.レイヤにて「N03-14_14_140401」を選択して、メニューから「編集モード」に切り替えます。  
![map012.png](/image/5c06409d-8764-29d8-9908-1a40c4e9cfa1.png)  
  
編集中のレイヤには鉛筆のアイコンが表示され、地図上の地物情報の色が変更されます。  
![map012.png](/image/95b50152-dcba-b368-8676-522d3d99c16a.png)  
  
6.CTRL+Vまたは「編集」→「地物の貼り付け」で編集中のレイヤに、コピーした地物情報を張り付けることができます。  
  
![map012.png](/image/1f2f0072-0ba2-5a74-7a56-cb4c46bb3556.png)  
  
7.レイヤから東京のレイヤを削除しても、張り付けた地物は残ることが確認できます。  
![map012.png](/image/627948c0-fa42-03b8-592d-b669bccdc339.png)  
  
![map012.png](/image/a484f721-0457-5165-d1c5-b825de187adc.png)  
  
### ジオメトリを簡素化する  
国土数値情報のジオメトリ情報は細かいため、データ量が多くなります。  
そこで、簡素化することで、データを削減します。  
  
1.「ベクタ」→「ジオメトリーツール」→「ジオメトリを簡素化する」を選択します。  
![map012.png](/image/842a032d-b333-cda7-8623-456f7e1a8a44.png)  
  
2.簡素化の許容範囲（ここでは0.001）と新規ファイルを入力してOKを押します。  
  
![map012.png](/image/b76697e6-63d8-08ff-4514-d813f7349ec4.png)  
  
3.変換がおわると新しいレイヤが作成されます。  
![map012.png](/image/1c816001-eb28-320b-133c-e287c431758d.png)  
  
### GeoJSONへの変換  
JavaScriptで使用できるようにGeoJSONに変換します。  
  
1.作成されたレイヤで右クリックして「名前を付けて保存」を選択します。  
  
  
2.形式と保存先を指定してOKを押します。  
![map012.png](/image/7eac2684-1104-588e-3d86-6fed338dcf26.png)  
  
これにより次のようなJSONファイルが作成されます  
  
```json
{
"type": "FeatureCollection",
"crs": { "type": "name", "properties": { "name": "urn:ogc:def:crs:EPSG::4612" } },
                                                                                
"features": [
{ "type": "Feature", "properties": { "N03_001": "神奈川県", "N03_002": null, "N03_003": "相模原市", "N03_004": "緑区", "N03_007": "14151" }, "geometry": { "type": "Polygon", "coordinates": [ [ [ 139.161601, 35.667907 ], [ 139.164979, 35.665279 ], [ 139.168358, 35.658556 ], [ 139.166952, 35.651066 ], [ 139.174948, 35.649889 ], [ 139.1790840000001, 35.645572 ], [ 139.189822, 35.646534 ], [ 139.194041, 35.651824 ], [ 139.198483, 35.653018 ], [ 139.202106, 35.653007 ], [ 139.213663, 35.647782 ], [ 139.215751, 35.645457 ], [ 139.214428, 35.641683 ], [ 139.215369, 35.637076 ], [ 139.221431, 35.629466 ], [ 139.219638, 35.625895 ], [ 139.225903, 35.623477 ], } },
// 略
```  
  
http://needtec.sakura.ne.jp/dc_example/tokyo_kanagawa.geojson  
  
## 作成したGeoJSONをブラウザで表示する。  
GeoJSONをブラウザに表示するにはD3.jsを使用します。  
  
以下からD3.jsをダウンロードしてください。  
http://ja.d3js.node.ws/  
  
作成したGeoJSONを表示するコードは以下のようになります。  
  
```js
// 参考
// http://shimz.me/blog/d3-js/2351
// http://shimz.me/blog/d3-js/2526
var width = 800;
var height = 600;
var vbox_x = 0;
var vbox_y = 0;
var vbox_default_width = vbox_width = 500;
var vbox_default_height = vbox_height = 500;

var projection = d3.geo.mercator()
   .center([139.700, 35.4500])
   .scale(30000)
   .translate([width / 2, height / 2]);

//geoJSONのデータをパスに変換する関数を作成
var path = d3.geo.path().projection(projection); 

//ステージとなるsvgを追加
var svg = d3.select("body").append("svg")
    .attr("width", width)
    .attr("height", height)
    .attr("viewBox", "" + vbox_x + " " + vbox_y + " " + vbox_width + " " + vbox_height); //viewBox属性を付加
 

//geoJSONファイルの読み込み
d3.json("tokyo_kanagawa.geojson", function(json) {
  return svg.append("svg:g")
            .attr("class", "tracts")
            .selectAll("path")
            .data(json.features)
            .enter()
            .append("svg:path")
            .attr("d", path)
            .attr("fill", "#ccc")
            .attr("stroke", "#000");
});

// ドラッグによる移動
var drag = d3.behavior.drag().on("drag", function(d) {
  vbox_x -= d3.event.dx;
  vbox_y -= d3.event.dy;
  return svg.attr("translate", "" + vbox_x + " " + vbox_y);
});
svg.call(drag);

// ズーム処理
zoom = d3.behavior.zoom().on("zoom", function(d) {
  var befere_vbox_width, before_vbox_height, d_x, d_y;
  befere_vbox_width = vbox_width;
  before_vbox_height = vbox_height;
  vbox_width = vbox_default_width * d3.event.scale;
  vbox_height = vbox_default_height * d3.event.scale;
  d_x = (befere_vbox_width - vbox_width) / 2;
  d_y = (before_vbox_height - vbox_height) / 2;
  vbox_x += d_x;
  vbox_y += d_y;
  return svg.attr("viewBox", "" + vbox_x + " " + vbox_y + " " + vbox_width + " " + vbox_height);  //svgタグのviewBox属性を更新
});
svg.call(zoom); 
```  
  
 **Demo**   
http://needtec.sakura.ne.jp/dc_example/simpleMap.html  
  
## 参考  
【D3.js】鶴舞う形の群馬県をSVGで描いてみる  
http://shimz.me/blog/d3-js/2351  
  
【D3.js】 viewBox属性を使ったPan/Zoon   
http://shimz.me/blog/d3-js/2526  
