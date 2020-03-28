# 東京メトロAPIを使ったアプリケーションを作ってみよう  
  
ここでは東京メトロAPIを使用してアプリケーションを開発する、方法について説明します。  
ここで記述しているものは、独自の調査により記述してあるので、公式での内容と異なることもありますので注意してください。  
  
ここでは実験コードをPHPで記述してますが、URL叩けばJSONで結果が帰ってくるので、任意の言語で開発できると思います。  
  
  
  
## 概要  
以下にその概要があります。  
https://developer.tokyometroapp.jp/info  
  
東京メトロにおける交通機関の情報をオープンデータとして公開するので、当該データを ~~テストする~~ 活用したアプリケーションの募集を行っています。  
  
ユーザー登録を行い、アクセストークンを入手することで以下のようなことがAPIを使用できます。  
  
・東京メトロの駅の情報を取得できます。駅の位置や、どのような施設が存在するか、どの路線に接続しているか調べることができます。  
・位置情報を検索キーにして出口や駅を検索したり、路線をキーに駅を検索したりできます。  
・各路線の線路の形を位置情報で取得したり、駅と駅の所用時間が取得できます。  
・遅延や運休などの運行状況が取得できます。  
・東京メトロ沿線での運賃がいくらかかるか取得できます。  
  
など  
  
## はじめよう  
１.以下の「開発サイト」からユーザー登録をします。登録まで数営業日かかるので、お茶でも飲んでゆっくり待ちましょう。  
  
https://developer.tokyometroapp.jp/info  
  
２.ユーザ登録が行われたら開発者サイトにログインをして、アクセストークンを追加します。  
  
![スクリーンショット 2014-10-10 21.02.19.png](/image/b77a4223-4c6a-9e79-e714-f56e0c873a16.png)  
  
 ***注意：ここで発行したアクセストークンは他人に漏れないようにしましょう。***   
  
３.APIの仕様にあるように特定のURLにアクセストークンを付与することで様々な情報がJSON形式で取得できます。  
  
  
 ***発行するURL***   
>https://api.tokyometroapp.jp/api/v2/datapoints?rdf:type=odpt:TrainInformation&acl:consumerKey=アクセストークン  
  
```json:取得結果
[{
  "@context":"http://vocab.tokyometroapp.jp/context_odpt_TrainInformation.json",
  "@id":"urn:ucode:_00001C000000000000010000030C3BE9",
  "dc:date":"2014-10-10T21:25:02+09:00","dct:valid":"2014-10-10T21:30:02+09:00",
  "odpt:operator":"odpt.Operator:TokyoMetro",
  "odpt:railway":"odpt.Railway:TokyoMetro.Tozai",
  "odpt:timeOfOrigin":"2014-10-08T15:35:00+09:00",
  "odpt:trainInformationText":"現在、平常どおり運転しています。",
  "@type":"odpt:TrainInformation"
},{
  "@context":"http://vocab.tokyometroapp.jp/context_odpt_TrainInformation.json",
  "@id":"urn:ucode:_00001C000000000000010000030C3BE7",
  "dc:date":"2014-10-10T21:25:02+09:00",
  "dct:valid":"2014-10-10T21:30:02+09:00",
  "odpt:operator":"odpt.Operator:TokyoMetro",
  "odpt:railway":"odpt.Railway:TokyoMetro.Marunouchi",
  "odpt:timeOfOrigin":"2014-09-29T11:00:00+09:00",
  "odpt:trainInformationText":"現在、平常どおり運転しています。",
  "@type":"odpt:TrainInformation"
},{
  //省略
}]
```  
  
「https://api.tokyometroapp.jp/api/v2/datapoints」に、以下のパラメータを指定することで、列車の運行状況を取得できます。  
  
|名前|説明|  
|:---|:---|  
|rdf:type|取得したいデータ種別。<BR>今回はodpt:TrainInformationを指定して列車運行情報を取得している|  
|acl:consumerKey|作成したアクセストークン|  
   
あとは、仕様書を見ながら格闘すればいいでしょう。  
  
## APIの基本的な話  
開発者サイトに行くと、API仕様書があります。  
以下の項目に目を通しておけば、大体のことはできると思います。  
  
 **API仕様書**   
https://developer.tokyometroapp.jp/documents  
 **鉄道情報**   
https://developer.tokyometroapp.jp/documents/railway  
 **地物情報**   
https://developer.tokyometroapp.jp/documents/facility  
  
### 二種類の検索方法  
検索には二種類存在しています。  
路線名や駅名といった特定の条件を元に検索する「データ検索API」と経度・緯度から検索を行う「地物情報取得API 」です。  
  
#### データ検索API  
以下のURLにパラメータを指定してデータを検索します。  
https://api.tokyometroapp.jp/api/v2/datapoints  
  
駅名や、路線名、IDなどを指定して検索が可能です。  
 **検索例：**   
>https://api.tokyometroapp.jp/api/v2/datapoints?rdf:type=odpt:Station&odpt:railway=odpt.Railway:TokyoMetro.Marunouchi&acl:consumerKey=APIキー  
  
この例では「路線が丸の内線であること」を検索条件にして駅情報を取得しています。  
  
データポイントではIDを取得してデータを特定して取得することも可能であす。  
このIDにはodptで始まるものを指定できます。  
>https://api.tokyometroapp.jp/api/v2/datapoints/odpt.Railway:TokyoMetro.Hanzomon?acl:consumerKey=APIキー  
  
この例では半蔵門線の情報を取得しています。  
  
#### 地物情報取得API   
以下のURLにパラメータを指定してデータを検索します。  
https://api.tokyometroapp.jp/api/v2/places  
  
経度と緯度を指定してデータを取得できます。  
  
  
 **検索例：**   
>https://api.tokyometroapp.jp/api/v2/places?rdf:type=odpt:Station&lat=35.6729562407498&lon=139.724074594678&radius=100&acl:consumerKey=APIキー  
  
この例では経度、緯度と範囲を指定して、その範囲の駅を取得しています。  
  
### 検索可能なrdf:type  
現在は以下のようなrdf:typeを取得できます。  
  
|rdf:type|datapoints|places|説明|  
|:-------|:---------|:-----|:--|  
|odpt:StationTimetable|○|×|駅時刻情報を取得できます。各駅の平日、土曜、休日の列車着時刻、発車時刻と急行や各駅などの種別を表します|  
|odpt:TrainInformation|○|×|運行状況を取得できます。１５分以上の遅延のさい更新されます。<BR>レスポンスのdct:validをみてデータを更新すると良いでしょう。また、情報の変化の有無はodpt:timeOfOriginが前回値と変わったかどうかでチェックするといいようです。<BR>ぶっちゃけ次のページと同様のデータです<BR>http://www.tokyometro.jp/unkou/index.html<BR>遅延情報の方とは一致しません|  
|odpt:Train|○|×|列車の在線位置を表します。どの駅にいるか、どの駅と駅の間にいるか、と５分以上の遅延を取得できます。列車のGPS情報はとれません。<BR>レスポンスのdct:validをみてデータを更新すると良いでしょう。<BR>また、非常時にこそ必要なデータですが、台風などの非常時には反映されないという事象も報告されています。<BR>https://developer.tokyometroapp.jp/forum/forums/1/topics/--57|  
|odpt:Station|○|○|駅の情報です。この情報はキャッシュしておき、ここに格納されているIDを使用して施設情報や乗降人数情報を取得するといいでしょう|  
|odpt:StationFacility|○|×|バリアフリー対応の駅のトイレやエレベータ、乗り換え可能路線、改札外の最寄り施設が取得できます。営業時間とかはのっていることはありますが、施設の位置情報は取得できません|  
|odpt:PassengerSurvey|○|×|駅毎の乗降人員数を１年単位で取得できます|  
|odpt:Railway|○|○|路線情報を取得します。運行系統名、駅間所要時間、駅間順序、路線の形状が取得できます。|  
|odpt:RailwayFare|○|×|２つ駅を指定して運賃を計算します。運賃にはICカードか切符、大人か子供かで変わります。<BR> fromStation,toStationを指定しての検索は部分一致です。しかし、料金を指定しての検索は範囲ではなく完全一致です。|  
|ug:Poi|○|○|地物情報。駅出入口情報を提供しています。|  
|odpt:TrainTimetable|○|×|特定の列車が其々の駅からいつ出発するかを格納している。<BR>startStationは他社始発の時のみしか格納されない。|  
  
## PHPでの実装例  
以下でPHPでの実装例を示します。  
slimを利用したシステムの一部なので、そのままコピーしても動きませんが、それなりに参考になると思います。  
  
```TokyoMetroApi.php
<?php
namespace MyLib;

/**
 * 東京メトロAPI実行用のクラス<br>
 * 各APIのレスポンスは以下の仕様に準拠している。<br>
 * @link https://developer.tokyometroapp.jp/documents/railway 駅情報ボキャブラリ仕様書
 * @link https://developer.tokyometroapp.jp/documents/facility 地物情報ボキャブラリ仕様書
 */
class TokyoMetroApi
{
    private $client;
    private $consumerKey;
    private $dataFolder;

    const END_POINT = 'https://api.tokyometroapp.jp/api/v2/';

    /** 結果コード：正常終了 */
    const RESULT_CODE_OK = 0;

    /** 結果コード：東京メトロAPIの異常 */
    const RESULT_CODE_ERR_API = 1;

    /** 東京メトロAPIの最大リトライ数 */
    const MAX_TRY_COUNT = 3;

    /** 東京メトロAPIのリトライ感覚 */
    const WAIT_COUNT = 100000; // 1ms

    /**
     * コンストラクタ
     * @param string $consumerKey 東京メトロAPIで登録したComsumerKey
     * @param string $dataFolder JSON情報の格納してあるフォルダ
     */
    public function __construct($consumerKey, $dataFolder)
    {
        $this->consumerKey = $consumerKey;
        $this->client = new \HTTP_Client();
        $this->dataFolder = $dataFolder;
    }

    public function getRailDirectionType() {
        return $this->readTypeJson($this->dataFolder . '/RailDirectionType.json');
    }
    public function getRailWayType() {
        return $this->readTypeJson($this->dataFolder . '/RailWayType.json');
    }
    public function getTrainOwnerType() {
        return $this->readTypeJson($this->dataFolder . '/TrainOwnerType.json');
    }
    public function getTrainType() {
        return $this->readTypeJson($this->dataFolder . '/TrainType.json');
    }

    private function readTypeJson($path) {
        $handle = fopen($path, 'r');
        $ret = fread($handle, filesize($path));
        fclose($handle);
        return (array)json_decode($ret);
    }

    /**
     * APIを実行してレスポンスの取得
     * もし、503エラーの場合は、WAIT_COUNTマイクロ秒後に
     * MAX_TRY_COUNT回までリトライする。
     * @param string $url 対象のURL
     * @param array $param パラメータの連想配列
     * @param int $trycount 現在の試行回数
     * @return array レスポンスの結果
     */
    private function getResponse($url, $param, $trycount)
    {
        $code = $this->client->get($url, $param);
        $res = $this->client->currentResponse();
        $body = null;
        if ($res) {
            if (isset($res['body'])) {
                $body = $res['body'];
            }
        }
        $ret = null;
        if ($code != 200) {
            // リトライ処理.
            // 100ms程度でブロックが掛かっているので,スリープして再実行.
            // https://developer.tokyometroapp.jp/forum/forums/1/topics/http-request-failed-http-1-1-503-service-unavailable
            if ($code == 503 and $trycount < TokyoMetroApi::MAX_TRY_COUNT) {
                usleep(($trycount + 1) * TokyoMetroApi::WAIT_COUNT);

                return $this->getResponse($url, $param, $trycount + 1);
            }

            $msg = sprintf('ResponceCode: %d Message:%s', $code, $body);
            $ret = array('resultCode' => TokyoMetroApi::RESULT_CODE_ERR_API,
                                     'errorMsg' => $msg,
                                     'contents' => null);
        } else {
            $ret = array('resultCode' => TokyoMetroApi::RESULT_CODE_OK,
                                     'errorMsg' => null,
                                     'contents' => json_decode($body));
        }

        return $ret;
    }

    /**
     * JSONを取得する。
     * @param string $url 対象のURL
     * @return array レスポンスの結果
     */
    public function getJson($url)
    {
        $param = array('acl:consumerKey' => $this->consumerKey);

        return $this->getResponse($url, $param, 0);
    }

    /**
     * データポイント用のAPIを実行する
     * @param array $param パラメータの連想配列
     * @return array レスポンスの結果
     */
    private function getDataPoints($param)
    {
        $url = TokyoMetroApi::END_POINT . 'datapoints';

        return $this->getResponse($url, $param, 0);
    }

    private function getPlaces($param)
    {
        $url = TokyoMetroApi::END_POINT . 'places';

        return $this->getResponse($url, $param, 0);
    }

    /**
     * 東京メトロ駅情報をすべて取得する
     * @return array レスポンスの結果
     */
    public function getStations()
    {
        $param = array('rdf:type' => 'odpt:Station',
                       'acl:consumerKey' => $this->consumerKey);

        return $this->getDataPoints($param);
    }

    /**
     * 東京メトロ駅情報を指定の条件で検索する.
     * <code>
     * $ctrl = new TokyoMetroApi(CONSUMER_KEY);
     * $param=array('odpt:railway' => 'odpt.Railway:TokyoMetro.Marunouchi');
     * $ret = $ctrl->findStation($param);
     * </code>
     * @param array $conditions 条件を指定した連想配列
     * @return array レスポンスの結果
     */
    public function findStation($conditions)
    {
        $param = array('rdf:type' => 'odpt:Station',
                       'acl:consumerKey' => $this->consumerKey);
        $param += $conditions;

        return $this->getDataPoints($param);
    }

    /**
     * 東京メトロ駅情報を位置情報の条件で検索する.
     * <code>
     * $ctrl = new TokyoMetroApi(CONSUMER_KEY);
     * $param=array('lat' => 35.6729562407498,
     *              'lon' =>139.724074594678,
     *              'radius' => 100);
     * $ret = $ctrl->findStation($param);
     * </code>
     * @param array $conditions 条件を指定した連想配列
     * @return array レスポンスの結果
     */
    public function findStationByPlace($conditions)
    {
        $param = array('rdf:type' => 'odpt:Station',
                       'acl:consumerKey' => $this->consumerKey);
        $param += $conditions;

        return $this->getPlaces($param);
    }
    /**
     * 東京メトロ路線情報を全て取得する.
     * @return array レスポンスの結果
     */
    public function getRailways()
    {
        $param = array('rdf:type' => 'odpt:Railway',
                       'acl:consumerKey' => $this->consumerKey);

        return $this->getDataPoints($param);
    }
    /**
     * 東京メトロ路線情報を指定の条件で検索する.
     * <code>
     * $ctrl = new TokyoMetroApi(CONSUMER_KEY);
     * $param=array('owl:sameAs' => 'odpt.Railway:TokyoMetro.Marunouchi');
     * $ret = $ctrl->findRailway($param);
     * </code>
     * @param array $conditions 条件を指定した連想配列
     * @return array レスポンスの結果
     */
    public function findRailway($conditions)
    {
        $param = array('rdf:type' => 'odpt:Railway',
                       'acl:consumerKey' => $this->consumerKey);
        $param += $conditions;

        return $this->getDataPoints($param);
    }
    public function findRailwayByPlace($conditions)
    {
        $param = array('rdf:type' => 'odpt:Railway',
                       'acl:consumerKey' => $this->consumerKey);
        $param += $conditions;

        return $this->getPlaces($param);
    }
    /**
     * 地物情報を使用して駅出入り口情報を取得する。
     * @return array レスポンスの結果
     */
    public function getPois()
    {
        $param = array('rdf:type' => 'ug:Poi',
                       'acl:consumerKey' => $this->consumerKey);

        return $this->getDataPoints($param);
    }

    /**
     * 地物情報を使用して駅出入り口情報を取得する。
     * <code>
     * $ctrl = new TokyoMetroApi(CONSUMER_KEY);
     * $ret = $ctrl->findPoi(array('@id'=>'urn:ucode:_00001C000000000000010000030C3EC1'));
     * if ($ret['resultCode'] != TokyoMetroApi::RESULT_CODE_OK) {
     *   sendJsonData($ret['resultCode'],  $ret['errorMsg'], null);
     *   exit();
     * }
     * $exit_info = $ret['contents'][0];
     * </code>
     * @param array $conditions 条件を指定した連想配列
     * @return array レスポンスの結果
     */
    public function findPoi($conditions)
    {
        $param = array('rdf:type' => 'ug:Poi',
                       'acl:consumerKey' => $this->consumerKey);
        $param += $conditions;

        return $this->getDataPoints($param);
    }
    public function findPoiByPlace($conditions)
    {
        $param = array('rdf:type' => 'ug:Poi',
                       'acl:consumerKey' => $this->consumerKey);
        $param += $conditions;

        return $this->getPlaces($param);
    }

    /**
     * 運賃を取得する
     * @return array レスポンスの結果
     */
    public function findFare($from, $to)
    {
        $param = array('rdf:type' => 'odpt:RailwayFare',
                                     'odpt:fromStation'=>$from,
                                     'odpt:toStation'=>$to,
                                     'acl:consumerKey' => $this->consumerKey);

        return $this->getDataPoints($param);
    }

    /**
     * 駅の施設に関する情報を取得する。
     * @return array レスポンスの結果
     */
    public function getStationFacilities() {
        $param = array('rdf:type' => 'odpt:StationFacility',
                       'acl:consumerKey' => $this->consumerKey);

        return $this->getDataPoints($param);
    }
    public function findStationFacility($conditions) {
        $param = array('rdf:type' => 'odpt:StationFacility',
                       'acl:consumerKey' => $this->consumerKey);
        $param += $conditions;
        return $this->getDataPoints($param);
    }

    /**
     *  列車運行情報 の取得
     * @return array レスポンスの結果
     */
    public function findTrainInformation($conditions) {
        $param = array('rdf:type' => 'odpt:TrainInformation',
                       'acl:consumerKey' => $this->consumerKey);
        $param += $conditions;
        return $this->getDataPoints($param);
    }
    /**
     *  列車ロケーション情報 の取得
     * @return array レスポンスの結果
     */
    public function findTrain($conditions) {
        $param = array('rdf:type' => 'odpt:Train',
                       'acl:consumerKey' => $this->consumerKey);
        $param += $conditions;
        return $this->getDataPoints($param);
    }

    /**
     * 駅乗降人員数を取得する。
     * @return array レスポンスの結果
     */
    public function getPassengerSurvey() {
        $param = array('rdf:type' => 'odpt:PassengerSurvey',
                       'acl:consumerKey' => $this->consumerKey);

        return $this->getDataPoints($param);
    }
}

```  
  
```RailDirectionType.json
{
  "odpt.RailDirection:TokyoMetro.Asakusa":"浅草 方面",
  "odpt.RailDirection:TokyoMetro.Ogikubo":"荻窪 方面",
  "odpt.RailDirection:TokyoMetro.Ikebukuro":"池袋 方面",
  "odpt.RailDirection:TokyoMetro.Honancho":"方南町 方面",
  "odpt.RailDirection:TokyoMetro.NakanoSakaue":"中野坂上 方面",
  "odpt.RailDirection:TokyoMetro.NakaMeguro":"中目黒 方面",
  "odpt.RailDirection:TokyoMetro.KitaSenju":"北千住 方面",
  "odpt.RailDirection:TokyoMetro.NishiFunabashi":"西船橋 方面",
  "odpt.RailDirection:TokyoMetro.Nakano":"中野 方面",
  "odpt.RailDirection:TokyoMetro.YoyogiUehara":"代々木上原 方面",
  "odpt.RailDirection:TokyoMetro.Ayase":"綾瀬 方面",
  "odpt.RailDirection:TokyoMetro.KitaAyase":"北綾瀬 方面",
  "odpt.RailDirection:TokyoMetro.ShinKiba":"新木場 方面",
  "odpt.RailDirection:TokyoMetro.Ikebukuro":"池袋 方面",
  "odpt.RailDirection:TokyoMetro.Oshiage":"押上 方面",
  "odpt.RailDirection:TokyoMetro.Shibuya":"渋谷 方面",
  "odpt.RailDirection:TokyoMetro.AkabaneIwabuchi":"赤羽岩淵 方面",
  "odpt.RailDirection:TokyoMetro.Meguro":"目黒 方面",
  "odpt.RailDirection:TokyoMetro.ShirokaneTakanawa":"白金高輪 方面",
  "odpt.RailDirection:TokyoMetro.Wakoshi":"和光市 方面",
  "odpt.RailDirection:TokyoMetro.KotakeMukaihara":"小竹向原 方面"
}

```  
  
```RailWayType.json
{
  "odpt.Railway:TokyoMetro.Ginza":"東京メトロ銀座線",
  "odpt.Railway:TokyoMetro.Marunouchi":"東京メトロ丸ノ内線",
  "odpt.Railway:TokyoMetro.Hibiya":"東京メトロ日比谷線",
  "odpt.Railway:TokyoMetro.Tozai":"東京メトロ東西線",
  "odpt.Railway:TokyoMetro.Chiyoda":"東京メトロ千代田線",
  "odpt.Railway:TokyoMetro.Yurakucho":"東京メトロ有楽町線",
  "odpt.Railway:TokyoMetro.Hanzomon":"東京メトロ半蔵門線",
  "odpt.Railway:TokyoMetro.Namboku":"東京メトロ南北線",
  "odpt.Railway:TokyoMetro.Fukutoshin":"東京メトロ副都心線"
}

```  
  
```TrainOwnerType.json
{
  "odpt.TrainOwner:TokyoMetro":"東京メトロ",
  "odpt.TrainOwner:Seibu":"西武鉄道",
  "odpt.TrainOwner:SaitamaRailway":"埼玉高速鉄道",
  "odpt.TrainOwner:Tobu":"東武鉄道",
  "odpt.TrainOwner:ToyoRapidRailway":"東葉高速鉄道",
  "odpt.TrainOwner:Toei":"都営地下鉄",
  "odpt.TrainOwner:Tokyu":"東急電鉄",
  "odpt.TrainOwner:JR-East":"JR東日本",
  "odpt.TrainOwner:Odakyu":"小田急電鉄"
}
```  
  
```TrainType.json
{
  "odpt.TrainType:TokyoMetro.Unknown":"不明",
  "odpt.TrainType:TokyoMetro.Local":"各停",
  "odpt.TrainType:TokyoMetro.Express":"急行",
  "odpt.TrainType:TokyoMetro.Rapid":"快速",
  "odpt.TrainType:TokyoMetro.SemiExpress":"準急",
  "odpt.TrainType:TokyoMetro.TamaExpress":"多摩急行",
  "odpt.TrainType:TokyoMetro.HolidayExpress":"土休急行",
  "odpt.TrainType:TokyoMetro.CommuterSemiExpress":"通勤準急",
  "odpt.TrainType:TokyoMetro.Extra":"臨時",
  "odpt.TrainType:TokyoMetro.RomanceCar":"特急ロマンスカー",
  "odpt.TrainType:TokyoMetro.RapidExpress":"快速急行",
  "odpt.TrainType:TokyoMetro.CommuterExpress":"通勤急行",
  "odpt.TrainType:TokyoMetro.LimitedExpress":"特急",
  "odpt.TrainType:TokyoMetro.CommuterLimitedExpress":"通勤特急",
  "odpt.TrainType:TokyoMetro.CommuterRapid":"通勤快速",
  "odpt.TrainType:TokyoMetro.ToyoRapid":"東葉快速"
}
```  
  
 ***単純な使用例***   
  
```使用例.php
<?php
require dirname(__FILE__) . '/../vendor/autoload.php';
const TOKYO_METRO_CONSUMER_KEY = 'APIキー';
const TOKYO_METRO_DATA_DIR = '../tokyometro_data';

$tmCtrl = new \MyLib\TokyoMetroApi(TOKYO_METRO_CONSUMER_KEY, TOKYO_METRO_DATA_DIR);

var_dump($tmCtrl->getPassengerSurvey());

var_dump($tmCtrl->getRailWayType());

// JSONファイルの内容をゲット
var_dump($tmCtrl->getRailDirectionType());
var_dump($tmCtrl->getTrainOwnerType());
var_dump($tmCtrl->getTrainType());

// 駅を調べる
var_dump($tmCtrl->getStations());
var_dump($tmCtrl->findStation(array('owl:sameAs' => 'odpt.Station:TokyoMetro.Tozai.Nakano')));

// 路線を調べる
var_dump($tmCtrl->getRailways());
var_dump($tmCtrl->findRailway(array('owl:sameAs' => 'odpt.Railway:TokyoMetro.Marunouchi')));

//　運賃を調べる
var_dump($tmCtrl->findFare('odpt.Station:TokyoMetro.Ginza.AoyamaItchome','odpt.Station:TokyoMetro.Chiyoda.Akasaka'));

// 出口情報
var_dump($tmCtrl->getPois());
var_dump($tmCtrl->findPoi(array('@id'=>'urn:ucode:_00001C000000000000010000030C3EC1')));

// 施設情報
var_dump($tmCtrl->getStationFacilities());
var_dump($tmCtrl->findStationFacility(array('@id'=>'urn:ucode:_00001C000000000000010000030C47F5')));

//列車のロケーション情報
var_dump($tmCtrl->findTrain(array()));

//駅を場所情報探す
var_dump($tmCtrl->findStationByPlace(array('lat' => 35.6729562407498,
                                           'lon' =>139.724074594678,
                                           'radius' => 100)));
//路線を場所情報で探す
var_dump($tmCtrl->findRailwayByPlace(array('lat' => 35.6729562407498,
                                           'lon' =>139.724074594678,
                                           'radius' => 100)));
//出入り口を場所情報で探す
var_dump($tmCtrl->findPoiByPlace(array('lat' => 35.6729562407498,
                                       'lon' =>139.724074594678,
                                       'radius' => 100)));

// 運行情報取得
var_dump($tmCtrl->findTrainInformation(array()));
$trans->updateCacheDb();

```  
  
  
### 実際のAPIの利用例  
以上のコードを利用すると次のようなアプリケーションが作成可能だと思います。  
実際の作成においては、実行結果をDBに格納したりしています。  
  
#### 駅出入口の確認  
ここでは東京メトロが提供する駅の出入り口の付近をストリートマップで確認できます。  
http://needtec.sakura.ne.jp/mtm/page/station_info?station=odpt.Station:TokyoMetro.Marunouchi.NakanoSakaue  
  
 ~~Googleと東京メトロAPIで位置情報が異なるので正確な情報が欲しい時は使うのが厳しいでしょう~~   
  
#### 列車運行状況の確認  
ここでは列車運行状況を確認できます。  
現在は、１５分以上の遅延が発生していなのでログが１つしかありませんが、列車が遅れれば、履歴として確認できるはずです。  
基本的に東京メトロにある運行状況と同じなります。  
  
http://needtec.sakura.ne.jp/mtm/page/train_info  
http://needtec.sakura.ne.jp/mtm/page/train_info?lang=en  
  
#### ロケーション情報を路線に表示する  
リアルタイムなロケーション情報を路線に描画します。  
  
http://needtec.sakura.ne.jp/mtm/page/railway_info  
  
応用として、過去のロケーション情報のプロットも行えます。  
  
http://needtec.sakura.ne.jp/mtm/page/subway_map  
  
※過去情報のプロットはメモリを食うので、現在はスマホだと落ちます。  
  
  
#### 運賃計算   
このアプリケーションでは２つの駅を指定して運賃を取得します。  
 ~~東京メトロAPIでなく駅すぱーとのAPI使ったほうが他の路線も計算できていいです。~~   
  
  
http://needtec.sakura.ne.jp/mtm/page/calculate_fare  
http://needtec.sakura.ne.jp/mtm/page/calculate_fare?lang=en  
  
## アプリケーション開発時の注意  
ここではアプリケーションを開発する際の注意について記述します。  
  
### サンプルが難しそうだが、実際はチョロイ  
公式の実装サンプルがrubyで、やってない人は、二の足を踏む方もいるかもしれませんが、実際はAPIキー付けてURL叩いてJSON解析するだけなので簡単です。  
とりあえず、APIキーをつけてURLをたたけばなんとかなります。  
  
サンプルコードは飾りです。偉い人にはわからんのです。  
  
### アクセストークンの管理  
アクセストークンは外部にもれないようにする必要があります。  
規約がかわってサーバーサイドを作成しないで、クライアントだけでの応募も可能になったようですが、そのアクセストークンの取り扱いには注意しなければなりません。  
たとえば逆コンパイルが容易な、.NETのクライアントアプリを作った場合、そのアクセストークンをどう隠蔽するか、良いアイディアは思いつきませんでした。  
  
### 開発者フォーラムの監視が必須  
怪しげな動作をしたり、API仕様書に一切書いてない仕様をサラっと述べたりするので開発者フォーラムの利用が必須になります。  
https://developer.tokyometroapp.jp/forum/forums/1  
  
この開発者フォーラムはMarkup言語が利用できるらしいので、Qiita程度の修飾はできるようです。  
しかしながら、検索ができないという致命的弱点があります。  
  
もちろん別途、バグ管理システムを立ち上げているわけではないので、トラブルが発生したら開発者は目視で開発者フォーラムを目を通し、関係のあるトピックが更新されたかどうかは手動でチェックするという苦行が必須になります。  
  
なお、自分に返信があったときはメールで通知がくるようです。  
  
#### いいぜ、てめえが検索させないっていうのなら、まずはそのふざけた幻想 をぶち殺す。  
Webスクレイピングによって検索ができるようにしました。  
ないよりマシです。  
  
 **東京メトロオープンデータサイト開発者サイト　操作ツール**   
https://github.com/mima3/tokyometro_dev  
  
怒られたら削除します。  
  
  
### レスポンスの文字コードはutf8(BOMなし)  
Content-Typeに文字コードの情報が入っていないので文字化けする環境があります。  
utf8(bomなし)が帰ってくることを前提に処理しましょう。  
  
### 「503:Your request rate is too high」が応答される  
短い期間でAPIを叩くと、この応答になります。  
同一IPアドレス+APIキーという条件にて100msのガードタイムがかかっています。間をあけてリトライしましょう。  
https://developer.tokyometroapp.jp/forum/forums/1/topics/http-request-failed-http-1-1-503-service-unavailable  
  
なお、この記事で示したPHPのコードは一定の感覚をおいて、数回リトライする作りになっています。  
この実装で、運賃情報のAPIを３万回ほど実行してエラーにはならなかったので、上記コードを参考にするといいかもしれません。  
  
  
追記  
  
https://developer.tokyometroapp.jp/forum/forums/1/topics/v2a-v2  
  
・v2とv2aでは独立してガードタイムを判定していない。v2,v2aでも同じタイミングでアクセスして条件を満たすとガードタイムが発動する。  
  
・ガードタイムは以下のすべてが一致した時に発動する。  
(1)IPが同じ  
(2)APIキーが同じ  
(3)rdf:typeが同じ  
  
どれか一つでも異なれば、ガードタイムは発動しない。つまり、複数端末から同じAPIキーでアクセスすることで処理の高速化がry  
  
### データの更新タイミング  
リアルタイムで更新される運行情報、列車ロケーションのレスポンスにはdct:validが付与されており、これをもとにデータの再取得をするといいでしょう。  
その他の情報については更新する可能性はありますが、機械的にそれを検知する方法はありません。開発者がニュース等を目視で監視して更新しろと公式では述べています。  
  
https://developer.tokyometroapp.jp/forum/forums/1/topics/--43  
  
駅情報とかをキャッシュしている場合、任意のタイミングで、それを更新するしくみを考えたほうがいいでしょう。  
  
### 遅延の概念は２つある。  
運行情報で取得できる情報は、東京メトロホームページの遅延情報と同じ物で、15分以上の遅延が出た場合にご案内が提示されます。  
http://www.tokyometro.jp/unkou/index.html  
  
一方、列車ロケーション情報は15分に達していない遅延に関しても取得できます。  
公式ページにある遅延証明書はおそらく、この情報を使用しています。  
  
https://developer.tokyometroapp.jp/forum/forums/1/topics/delay  
  
  
### 位置情報はGoogleMapとずれている。  
以下の検証用アプリはug:Poiで取得した情報を利用して出入り口をマークしていますが、GoogleMapと結構ずれています。  
http://needtec.sakura.ne.jp/mtm/station_map  
  
おそらく、シビアな位置情報の一致が必要なアプリケーションの作成にはリスクがあるでしょう。  
  
国土地理院の地図で確認すると、それっぽいところに表示されているので、なんらかの差異があるようです。  
http://vldb.gsi.go.jp/sokuchi/surveycalc/tky2jgd/main.html  
  
ちなみに世界測地系と日本測地系の違いかとも思いましたが、変換すると余計ずれるので、そうではなさげです。  
  
  
### 検索の方法が部分一致のものがある。  
以下の例はowl:sameAsを検索条件に路線情報を取得するものです。  
  
>https://api.tokyometroapp.jp/api/v2/datapoints?rdf:type=odpt:Railway&owl:sameAs=odpt.Railway:TokyoMetro.Marunouchi&acl:consumerKey=XXXX  
  
普通、この例では「odpt.Railway:TokyoMetro.Marunouchi」の結果のみが取得されると思いますが、実際は「odpt.Railway:TokyoMetro.Marunouchi」と「odpt.Railway:TokyoMetro.MarunouchiBranch」が取得されます。  
  
これは、検索を部分一致でしている場合があるからです。  
  
 ***運賃を取得するときの、odpt:fromStation、odpt:toStationは部分一致なので、レスポンスデータの取り扱いを間違うと予期せぬ料金になります***   
  
### 列車ロケーション情報で特定タイミングの特定路線のデータが取得できない疑義がある。  
  
https://developer.tokyometroapp.jp/forum/forums/1/topics/startingstation-terminalstation-fromstation-tostation  
  
検証した結果、5:00-6:00 , 7:00-10:00の間で方南町を含むstartingStation、terminalStation、fromStation、toStationのデータがAPI側から配信されていない。すくなくとも、startingStation、terminalStationについては配信していない疑義は濃厚です。  
  
当然、リアルタイムのデータ扱うのはバグがでやすいので、他にもある可能性はあります。  
  
追記 11/09:  
  
この事象自体はなおったようですが、列車ロケーション情報の挙動自体は、くっそ怪しいです。  
  
一応,11/2時点で関連バグをまとめましたが、運営様におきましては、イシュー管理には、一切の興味がないようですので、まぁ、しかたないね。  
  
https://developer.tokyometroapp.jp/forum/forums/1/topics/11-2  
  
  
 ***つまり、列車ロケーション情報を使用した場合、元データを検証用にしばらく保存しておく仕組みにしたほうがいいです。***   
  
### おまえはいつから、全てのトイレの情報を教えてもらえるとおもっていた？  
施設情報で取得できるトイレの情報は、レスポンスデータにバリアフリーとあるように車椅子のトイレだけです。  
  
https://developer.tokyometroapp.jp/forum/forums/1/topics/--80  
  
車椅子乗っていない人は漏らすか、自分で頑張って探してください。  
  
### odpt:TrainTimetableの時刻には出発時刻と到達時刻がある。  
odpt:TrainTimetableの時刻には出発時刻と到達時刻がある  
  
終着駅は到着時刻を表し、arrivalTime,arivalStationとなっており、それ以外は出発時刻を表しdepartureTime、departureStationとなる。  
  
2014/10/27現在、以下のようなデータが取得できる。  
  
```json
[{
  "@context":"https://vocab.tokyometroapp.jp/context_odpt_TrainTimetable.jsonld",
  "@id":"urn:ucode:_00001C000000000000010000030D0AA5",
  "@type":"odpt:TrainTimetable",
  "owl:sameAs":"odpt.TrainTimetable:TokyoMetro.Marunouchi.A11321.Weekdays",
  "odpt:trainNumber":"A11321",
  "odpt:railway":"odpt.Railway:TokyoMetro.Marunouchi",
  "odpt:operator":"odpt.Operator:TokyoMetro",
  "dc:date":"2014-10-24T21:56:21+09:00",
  "odpt:trainType":"odpt.TrainType:TokyoMetro.Local",
  "odpt:railDirection":"odpt.RailDirection:TokyoMetro.Honancho",
  "odpt:weekdays":[
    {
      "odpt:departureTime":"11:04",
      "odpt:departureStation":"odpt.Station:TokyoMetro.MarunouchiBranch.NakanoSakaue"
    },{
      "odpt:departureTime":"11:07",
      "odpt:departureStation":"odpt.Station:TokyoMetro.MarunouchiBranch.NakanoShimbashi"
    },{
      "odpt:departureTime":"11:08",
      "odpt:departureStation":"odpt.Station:TokyoMetro.MarunouchiBranch.NakanoFujimicho"
    },{
      "odpt:arrivalTime":"11:11",
      "odpt:arrivalStation":"odpt.Station:TokyoMetro.MarunouchiBranch.Honancho"
    }
  ],
  "odpt:train":"odpt.Train:TokyoMetro.Marunouchi.A11321",
  "odpt:terminalStation":"odpt.Station:TokyoMetro.MarunouchiBranch.Honancho",
  "odpt:trainOwner":"odpt.TrainOwner:TokyoMetro"
}]
```  
  
  
  
### 施設情報のspac:hasAssistantの説明。  
トイレ内のバリアフリー施設をあらわすspac:hasAssistantには次の値が入る可能性があります。  
  
ug:BabyChangingTable　：　おむつ交換台  
  
spac:WheelchairAssesible　：　車いす対応  
  
ug:BabyChair　：　ベビーチェア　用を足している時に赤ん坊を座らせておく椅子  
  
ug:ToiletForOstomate　：　オストメイト　人工肛門・人工膀胱を造設した人が使えるトイレ  
  
  
### 列車運行情報のデータ更新の検知方法  
列車運行情報はodpt:timeOfOriginというデータ発生時刻を持っています。  
通常は、これが変化したらデータが更新されたと見なしていいかもしれませんが、特定の路線で３５日以上、運行情報が更新されず履歴が亡くなった場合に、データを取得すると取得のたびodpt:timeOfOriginは更新されます。  
  
これは下記の不具合に対応した際、過去履歴情報が空の場合情報取得時点でのタイムスタンプを挿入しているためです。  
  
https://developer.tokyometroapp.jp/forum/forums/1/topics/api--8  
  
変化を検知するにはodpt:trainInformationTextが前回値と変わったかどうかをチェックした方が良いでしょう。  
  
なお、odpt:trainInformationStatusはだめです。odpt:trainInformationStatusが同じで、odpt:trainInformationTextが変わる場合があります。  
  
### 多言語対応  
公式にはやってません。  
自分でがんばりましょう。  
  
コストを掛けずにやるには、２００万文字までは無料で使用できるMsTranslatorが良いと思います。  
なお、方南町を「How to Minami-Cho」とか訳すので、金があるならgoogle翻訳、使うなりしましょう。  
ちなみに駅情報、路線情報、施設情報の結果を、必要そうな箇所を翻訳しただけで、５万文字程度でしたので、英語以外も対応できるでしょう。  
ただし、機械翻訳は遅いので、大量のデータを一気に翻訳しないように工夫が必要です。  
  
### 不安定さ  
絶賛開発中という感じがするので、現時点でこのAPIを前提にした、止まると洒落に成らないシステムを構築するのはやめておいたほうがいいでしょう。  
（そもそも免責事項にはいっていた気もします）  
  
また、デグレを起こしたり、修正により、別のAPIのデータが不正になるなど、おそらく、開発プロセスに回帰テストは含まれていません。  
APIがなにか回収した場合は、基本、アプリ側で回帰テストをする必要があるでしょう。  
  
### 予告なしで挙動が変わる前提でデバッグすること  
わりと予告なしにリリースして、挙動を変えた数時間後にニュースに表示されることがあります。  
  
そして、その数時間の間は、挙動の変わったアプリケーションのデバッグを実に無駄に行うはめになることがあります。  
  
開発者は開発フォーラムやニュースに一切、書き込みがなくても、APIそのものの挙動が突然変わる可能性も考慮しつつ、デバッグする必要があります。  
  
### 応募時の注意  
 ~~コンテストに応募をしないアプリケーションの存在を許さないオープンデータの定義に挑戦するオープンデータなので、11/17までに嫌でも応募しましょう。~~   
  
 ~~その際、5分を上限としたYoutubeの動画を作成が **必須** になります。~~   
  
 ~~また、スマホの人はGoogle Play、App Store、Windowsストアの登録が必須なので、ギリギリはまずいです。~~   
  
  
 **2015年4月以降もAPIの存続がきまり、4月以降に新規APIキーの発行が可能になると発表がありました。当然、アプリケーションコンテストに参加しないアプリの存在もみとめてもらえます。**   
  
  
### odpt:Stationで取得するodpt:connectiongRailwayと公式ページの「駅の情報・路線図」に食い違いがある  
公式ページの「[駅の情報・路線図](http://www.tokyometro.jp/station/index02.html "駅の情報・路線図")」とodpt:Stationで取得するodpt:connectiongRailwayにいくつか食い違いがあります。  
  
この詳細は下記の通りです。  
  
http://needtec.sakura.ne.jp/doc/connectingRailwayBug_20141113.xlsx  
  
わたしはこのバグを18日まで報告する気はないので、もし致命的だと考えるひとがいたら、ご自由にお使いください。  
  
### 各停運転区間の停車駅では、駅時刻表の列車タイプは必ず各駅停車になる。それが列車としては急行であっても！  
列車時刻表の列車タイプ（各駅停車や急行）は直観通りの意味です。  
  
ただし、駅時刻表の列車タイプには注意が必要です。  
  
急行や各駅停車がまざる区間の駅時刻表の列車タイプは急行や各停が入ります  
  
しかし、各停運転区間の駅時刻表の列車タイプは必ず各駅停車になります。  
  
https://developer.tokyometroapp.jp/forum/forums/1/topics/odpt-traintype-tokyometro-rapid  
  
フォーラムより抜粋した例  
  
```
例）
列番：B695SR 　種別：通勤快速
西船橋　定刻　６時３３分発　駅時刻表：odpt.TrainType:TokyoMetro.CommuterRapid
浦安　　定刻　６時４０分発　駅時刻表：odpt.TrainType:TokyoMetro.Local
飯田橋　定刻　７時　７分発　駅時刻表：odpt.TrainType:TokyoMetro.Local

列番：B964TR　種別：快速
浦安　　定刻　９時４９分発　駅時刻表：odpt.TrainType:TokyoMetro.Rapid
飯田橋　定刻１０時１１分発　駅時刻表：odpt.TrainType:TokyoMetro.Local
```  
  
この仕様は、公式の時刻表と合わせるための、仕様です。  
  
http://www.tokyometro.jp/station/iidabashi/timetable/tozai/b/index.html  
  
### いいぜ、運営、お前が、締切３日前にAPIをリリースするっていうなら、まずは、そのふざけた機能をテストするアプリを応募してやる。  
  
「締切３日前にリリースされたAPIの挙動を確認するアプリ」  
  
http://needtec.sakura.ne.jp/mtm_alpha/?lang=ja  
  
### 中野坂上駅のIDがカオス。  
中野坂上は、2種類のsameAsがあります。「odpt.Station:TokyoMetro.MarunouchiBranch.NakanoSakaue」と「odpt.Station:TokyoMetro.Marunouchi.NakanoSakaue」です。  
そして、これらの使い分けは統一しようと頑張ったけどダメでしたという感じです。  
  
丸の内分岐線を走る列車時刻表ではodpt.Station:TokyoMetro.MarunouchiBranch.NakanoSakaueを使用しています。  
しかしながら、列車ロケーション情報ではodpt.Station:TokyoMetro.Marunouchi.NakanoSakaueになっています。  
https://developer.tokyometroapp.jp/errata  
  
そして、駅時刻表の到着駅にかんしても、odpt.Station:TokyoMetro.Marunouchi.NakanoSakaueになっています。  
  
ヒャッハー、中野坂上は本当に地獄だぜ！  
  
### 副都心線と有楽町線の和光市へ向かう一部区間の駅時刻表の行先がごっちゃになっている。  
和光市へむかう駅時刻表の行先の駅のsameAsはodpt.Station:TokyoMetro.Yurakucho.Wakoshiかodpt.Station:TokyoMetro.Fukutoshin.Wakoshiの二つあります。  
  
んで例によって駅時刻表の終着駅と列車時刻表の終着駅で、ごちゃごちゃにつかっているので正常にリンクできません。  
  
どうも、駅時刻表は何も考えずに、odpt.Station:TokyoMetro.Yurakucho.Wakoshiをつかっているようです。  
  
和光市～池袋を仲良くはしる有楽町線と副都心線・・・おそろしい子・・・  
  
### 列車番号とはいったいなんなのだ  
列車番号は物理的な車両を表すものではないと推測できます。  
  
以下の例を見てください。  
  
```
https://api.tokyometroapp.jp/api/v2/datapoints?rdf:type=odpt:StationTimetable&odpt:station=odpt.Station:TokyoMetro.MarunouchiBranch.NakanoFujimicho&acl:consumerKey=(APIキー
```  
  
中野富士見町から荻窪行の列車が出ているのが確認できます。  
  
これは時刻表的に正しいです。  
http://www.tokyometro.jp/station/nakano-fujimicho/timetable/marunouchi/zb/  
  
列車時刻表から逆引きすると、この列車の列車番号B05371です。B05371は中野坂上で列車番号A0537となり、荻窪へ向かいます。  
  
つまり、同じ列車ですが、列車番号は異なるのです。  
  
列車番号とはいったいなんなのか・・・いまのぼくには理解できない。  
  
そして駅時刻表と列車時刻表の行先が一致しているか確認したのが下記のとおり。  
http://needtec.sakura.ne.jp/doc/TokyoMetroTimeTableBug.xlsx  
  
・有楽町線、副都心線の共通線路で,Fukutoshin.Wakoshiが終着駅の列車に対して駅時刻表の行先が.Yurakucho.Wakoshiとなる  
・中野坂上のIDがBranchつけるか否かが統一されていない。  
・中野富士見町発、列車番号B05371は中野坂上で列車番号A0537となり、荻窪へ向かう（これは仕様だとおもわれる）  
