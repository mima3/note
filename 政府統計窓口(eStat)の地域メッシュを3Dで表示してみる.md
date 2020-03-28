以前、政府統計窓口の地域メッシュをWebブラウザで表示しました。  
  
 **政府統計窓口(eStat)の地域メッシュをWebブラウザで表示する方法**   
http://qiita.com/mima_ita/items/38784095a146c87dcd23  
  
せっかくなので、three.jsを用いて地域メッシュを3Dで表示してみます。  
  
 **demo**  
http://needtec.sakura.ne.jp/threemap/mesh3d_example.html  
  
 **ソースコード**  
https://github.com/mima3/threemap  
  
![3dmesh.png](/image/61f38d87-abf0-b71a-dd7e-fe031bd78ad1.png)  
  
  
## 指定範囲のデータをJSONで取得できるようにする。  
政府統計窓口の地域メッシュと、国土数値情報の行政区域を範囲を指定して取得できるようにします。  
  
「細かいことはいいんだよ！俺は３Dで表示したいんだ！」って人は、以下の呼び出し例をコピペすればJSONが取得できるので、それを利用してください。  
  
### 指定の範囲の地域メッシュをJSONで取得できるようにする。  
まず、政府統計窓口の地域メッシュをHTTP経由でJSONで取得できるようにします。  
  
 **呼び出し例：**   
http://needtec.sakura.ne.jp/estat/json/get_mesh_stat_group_by_mesh?swlat=35.45&swlng=139.4&nelat=35.7&nelng=139.81&stat_id=T000608  
  
 **取得内容：**   
  
```json
{
  "type": "FeatureCollection", 
  "features": [
    {
      "geometry": {
        "type": "Polygon", 
        "coordinates": [
          [
            [139.4124999999999, 35.45833333333333], 
            [139.4249999999999, 35.45833333333333], 
            [139.4249999999999, 35.46666666666666], 
            [139.4124999999999, 35.46666666666666], 
            [139.4124999999999, 35.45833333333333]
          ]
        ]
      }, 
      "type": "Feature", 
      "properties": {
        "世帯総数": "5018", 
        "女": "6093", "\u4eba\u53e3\u7dcf\u6570": "12349", 
        "男": "6256", "area": "53391353"
      }
    },// 略 
  ]
} 
```  
  
 **ソースコード**   
https://github.com/mima3/estat  
  
この実装はPython+bottleで行っています。  
  
### 指定の範囲の行政区域をGeoJSONで取得できるようにする。  
同様に国土数値情報の行政区域も範囲を指定してGeoJSONで取得できるようにします。  
  
 **呼び出し例：**   
http://needtec.sakura.ne.jp/kokudo/json/get_administrative_district_by_geometry?swlat=35.45&swlng=139.4&nelat=35.7&nelng=139.81  
  
 **取得内容：**   
  
```json
{
  "type": "FeatureCollection", 
  "features": [
    {
      "geometry": {
        "type": "Polygon", 
        "coordinates": [
          [
            [139.4478029999999, 35.42797099999999], 
            [139.445154, 35.428581], // 略
          ]
        ]
      }, 
      "type": "Feature", 
      "properties": {
        "prefectureName": "\u795e\u5948\u5ddd\u770c", 
        "administrativeAreaCode": "14218", 
        "cityName": "\u7dbe\u702c\u5e02", 
        "countryName": "                    ", 
        "subPrefectureName": "                    ", 
        "PK_UID": 94566
      }
    }, // 略
  ]
}
```  
  
 **ソースコード**   
https://github.com/mima3/kokudo  
  
この実装はPython+bottleで行っています。  
  
## three.jsを使用して3Dで表示する  
### D3.jsで読み込んだ行政区域のGeojsonを3D表示する。  
GeojsonをD3.jsで読み込み、作成されたSVGをthree.jsで3D表示します。  
この方法は、下記のページを参考にしました。  
  
 **GUNMA GIS GEEK D3.jsで読み込んだgeoJSONデータをthree.jsを使って3D表示する。**   
http://shimz.me/blog/d3-js/3137  
  
 **Render geographic information in 3D with Three.js and D3.js**   
http://www.smartjava.org/content/render-geographic-information-3d-threejs-and-d3js  
  
まず、JSONをD3.jsを利用して取得します。  
この際、[async.js](https://github.com/caolan/async "async.js")を用いて並列でJSONをリクエストしています。  
  
```js
async.parallel([
  function (callback) {
    d3.json("http://needtec.sakura.ne.jp/kokudo/json/get_administrative_district_by_geometry?swlat=35.45&swlng=139.4&nelat=35.7&nelng=139.81", function(json) {
        callback(null, json);
    });
  },
  function (callback) {
    d3.json("http://needtec.sakura.ne.jp/estat/json/get_mesh_stat_group_by_mesh?swlat=35.45&swlng=139.4&nelat=35.7&nelng=139.81&stat_id=T000608", function(json) {
        callback(null, json);
    });
  }
], function(err, results) {
  // 略
});
```  
  
この取得したgeojsonをsvg pathに変換して、その後、three.jsに表示します。  
  
```js
  function initScene() {
    container = $('#map');

    //シーンの追加
    scene = new THREE.Scene();

    //カメラの設定
    camera = new THREE.PerspectiveCamera(70, window.innerWidth / window.innerHeight, 1, 10000);
    camera.position.set(-6.005123939520757, -6.754277108005312, 3.1756149676051133);
    var target = new THREE.Vector3();
    target.x = -7.162140160089945;
    target.y = 0.42803104270646924;
    target.z = -2.063747120298727;
    //camera.lookAt(target); // TrackballControls使用中は効かない

    scene.add(camera);

    //ライティングの設定
    var light = new THREE.DirectionalLight(0xffffff, 2);
    light.position.set(-10, -20, 30).normalize();
    scene.add(light);

    light2 = new THREE.AmbientLight(0x333333);               
    scene.add(light2);  

    var projection = d3.geo.equirectangular()
       .scale(900)
       .translate([-2200, 560]);

    //上下が反転しているのでムリくり合わせる
    var reverseProjection = function(x, y) {
      pt = projection(x, y);
      pt[1] *= -1;
      return pt;
    };

    //geoJSONのデータをパスに変換する関数を作成
    var path = d3.geo.path().projection(reverseProjection);

    createAdministrativeDistrictMesh(results[0].features);
    function createAdministrativeDistrictMesh(geodata) {
      //geoJSON→svg path→three.js mesh変換    
      var countries = [];
      for (i = 0 ; i < geodata.length ; i++) {
        var geoFeature = geodata[i];
        var properties = geoFeature.properties;
        var feature = path(geoFeature);

        //svgパスをthree.jsのmeshに変換
        var gmesh = transformSVGPathExposed(feature);
        //console.log(properties, gmesh);
        countries.push({"data": properties, "mesh": gmesh});
      }

      //mesh追加
      for (i = 0 ; i < countries.length ; i++) {
      
          var material = new THREE.MeshLambertMaterial({color:0x888888});
          
          var shape3d = countries[i].mesh.extrude({
              amount: 0,
              bevelEnabled: false
          });

          var toAdd = new THREE.Mesh(shape3d, material);
          //toAdd.rotation.x = 60;
          scene.add(toAdd);
      }
    }
```  
  
svg pathからthree.jsへの変換は下記のライブラリを使用します。  
  
 **d3-threeD**   
https://github.com/asutherland/d3-threeD  
  
このライブラリにはtransformSVGPathという内部関数があります。この関数がsvg pathからthree.jsのジオメトリに変換します。  
  
この関数は内部関数のため、d3-threeD.jsを以下のように変更して外部から利用できるようにします。  
  
**d3-threeD.js**  
```js:d3-threeD.js
/* This Source Code Form is subject to the terms of the Mozilla Public
 * License, v. 2.0. If a copy of the MPL was not distributed with this file,
 * You can obtain one at http://mozilla.org/MPL/2.0/. */
var transformSVGPathExposed;

function d3threeD(exports) {
// 略
transformSVGPathExposed = transformSVGPath;
}
```  
  
  
### 地域メッシュをBoxGeometryで表現する  
地域メッシュのGeojsonからBoxGeometryを作成します。  
geojsonの経度緯度をBoxGeometryのx,yに指定し、男、女の人口数をz軸に指定しています。  
  
  
```js
    function createPopulationMesh(geodata) {
      //geoJSON→svg path→three.js mesh変換    
      var countries = [];
      for (i = 0 ; i < geodata.length ; i++) {
        var geoFeature = geodata[i];
        console.log();
        var properties = geoFeature.properties;

        var extX = d3.extent(geoFeature.geometry.coordinates[0], function(d) { return d[0];});
        var extY = d3.extent(geoFeature.geometry.coordinates[0], function(d) { return d[1];});
        var ptMin = reverseProjection([extX[0], extY[0]]);
        var ptMax = reverseProjection([extX[1], extY[1]]);

        var vm = parseFloat(geoFeature.properties['男']) /10000;
        var vw = parseFloat(geoFeature.properties['女']) /10000;

        var geometry = new THREE.BoxGeometry(Math.abs(ptMax[0] - ptMin[0]), Math.abs(ptMax[1] - ptMin[1]), vm);
        var material = new THREE.MeshLambertMaterial({
          color:0x0000ff,
          transparent: true,
          opacity: 0.7
        });
        var mesh = new THREE.Mesh(geometry, material);
        mesh.position.set(ptMin[0], ptMin[1], vm/2);
        scene.add(mesh);

        var geometry = new THREE.BoxGeometry(Math.abs(ptMax[0] - ptMin[0]), Math.abs(ptMax[1] - ptMin[1]), vw);
        var material = new THREE.MeshLambertMaterial({
          color:0xff0000,
          transparent: true,
          opacity: 0.7
        });
        var mesh = new THREE.Mesh(geometry, material);
        mesh.position.set(ptMin[0], ptMin[1], vm + (vw/2));
        scene.add(mesh);
      }
    }
```  
  
### まとめ  
three.jsを使用すれば簡単に3D表現ができます。  
D3.jsを使用してgeojsonをSVGに簡単に変換できます。  
D3.jsで作成したSVGをthree.jsで使用できるようにするには、d3-threeD.jsの内部関数、transformSVGPathを使用します。  
  
~~地域メッシュは３Dで表現するより色で表現した方がわかりやすいってはっきりわかんだね。~~  
  
