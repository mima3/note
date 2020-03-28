このたび、東京メトロのオープンデータと福岡市のオープンデータを無理やり多言語対応しました。  
  
 **地下鉄多言語化MOD**   
https://developer.tokyometroapp.jp/app/15  
  
 **福岡市オープンデータ多言語化MOD**   
http://needtec.sakura.ne.jp/fukuoka_map  
  
この記事では、なるべく、楽にWebアプリケーションを多言語対応する方法について考察したいと思います。  
  
## 機械翻訳の利用  
MicrosoftTranslatorを用いることで、月あたり200万文字の機械翻訳がおこなえます。  
  
 **PHPでMicrosoft Translate APIの翻訳機能を使ってみる**   
http://www.arubeh.com/archives/783  
  
 **Translator Language Codes**   
対応言語のコード一覧  
http://msdn.microsoft.com/en-us/library/hh456380.aspx  
  
 **bing翻訳**   
MsTranslatorを使用している翻訳サイト  
http://www.bing.com/translator/  
  
基本、これを使用することになるのですが２つ問題があります。  
  
１つは、速度です。APIのレスポンスは相当に遅いです。  
２つ目は機械翻訳の精度です。  
  
機械翻訳をそのままリリースして、東北観光博は新聞沙汰になったことは記憶に新しいことだと思います。  
  
 **東北観光博機械翻訳騒動～翻訳者の視点**  
http://togetter.com/li/288496  
  
  
これに対応するためにアプリケーションが直接、MicrosoftTranslatorで翻訳を行うのでなく、データベースを経由するようにします。  
  
## データベースを利用した機械翻訳の利用  
![翻訳の仕組み.png](/image/f7d6f5c6-39e6-90e7-cbe8-5e4ef443de7d.png)  
  
翻訳結果をデータベースに保存することで外部APIへのアクセスを減らしパフォーマンスを向上させるのと同時に、人による翻訳の修正もしくは、あらかじめ翻訳結果を登録しておくことができます。  
  
結局、人手を介すなら機械翻訳使わなくてもいいじゃないかという意見もあるでしょうが、翻訳情報が存在しない場合に機械翻訳を利用するにはいくつかの利点があります。  
  
(1)人手による翻訳ができない場合に最低限の翻訳情報の提供。これは、資金がたりなくて翻訳できない場合もそうですが、時間的余裕がない場合にも有効です。たとえば、先の東京メトロオープンデータの場合、駅の施設情報は人による翻訳情報を提供するチャンスはありますが、運行情報などは、時間的に翻訳する余裕はありません。  
  
(2)開発初期における最低限の翻訳の提供。開発初期において、不正確でいいから翻訳情報が欲しい場合があります。たとえばオランダ語などは、かなり長い文章になったり、アラビア語は右から読んだりしますが、そういう想定が開発中につくのは非常に大きいアドバンテージです。  
よく、英語だけで多言語対応を開発していて、開発の末期で別の言語固有のバグが出てくるというのを防ぐ効果があると思います。  
  
## 具体的なPHPの実装  
今回作ったコードを以下に公開しています。  
https://github.com/mima3/MultilanguageTokyoMetro  
  
※ただし、東京メトロAPIのアクセストークンの配布が停止されたので動作しません。  
  
翻訳に関係する主なコードは下記のとおりです。  
  
 **MultilanguageTokyoMetro/src/Model/MsTranslatorCacheModel.php**   
このコードは翻訳情報をキャッシュするためのモデルです  
  
```php
<?php
namespace Model;

/**
 * Microsoft Translatorの結果をキャッシュするモデル<br>
 */
class MsTranslatorCacheModel extends \Model\ModelBase
{
    /**
     * データベースの設定を行う.
     */
    public function setup()
    {
        $this->db->exec(
            "CREATE TABLE IF NOT EXISTS translator_cache (
              id INTEGER PRIMARY KEY AUTOINCREMENT,
              lang TEXT,
              src TEXT,
              result TEXT,
              updated TIMESTAMP DEFAULT (DATETIME('now','localtime')),
              author TEXT,
              UNIQUE(lang, src)
            );"
        );
        $this->db->exec(
            "CREATE INDEX IF NOT EXISTS translator_cache_lang_index ON  translator_cache(lang);"
        );
        $this->db->exec(
            "CREATE INDEX IF NOT EXISTS translator_cache_src_index ON  translator_cache(lang,src);"
        );
    }

    /**
     * 特定の言語のキャッシュを取得する
     * @param  string                                                 $lang 言語
     * @return キャッシュの内容.ない場合はnullとなる.
     */
    public function getCache($lang)
    {
        $ret = \ORM::for_table('translator_cache')
            ->where_equal('lang', $lang)
            ->find_array();
        $pairs = array();
        foreach ($ret as $r) {
            $pairs += array($r['src']=>$r['result']);
        }

        return $pairs;
    }

    /**
     * キャッシュを登録する
     * @param string $lang  言語
     * @param array  $pairs srcとresultのペアの一覧
     */
    public function addCache($lang, $pairs)
    {
        $this->db->beginTransaction();
        $updated = time();

        foreach ($pairs as $src => $result) {
            $row = \ORM::for_table('translator_cache')->create();
            $row->lang = $lang;
            $row->src = $src;
            $row->result = $result;
            $row->updated = $updated;
            $row->save();
        }

        $this->db->commit();
    }
    public function deleteCache($lang)
    {
        \ORM::for_table('translator_cache')
            ->where_equal('lang', $lang)
            ->delete_many();
    }

    private function createCond($id, $lang, $src, $result, $author)
    {
        $cond = \ORM::for_table('translator_cache');
        if ($id) {
            $cond = $cond->where_equal('id', $id);
        }
        if ($lang) {
            $cond = $cond->where_equal('lang', $lang);
        }
        if ($src) {
            $cond = $cond->where_like('src', '%' . $src .'%');
        }
        if ($result) {
            $cond = $cond->where_like('result', '%' . $result . '%');
        }
        if ($author) {
            $cond = $cond->where_like('author', '%' . $author . '%');
        }

        return $cond;
    }

    /**
     * 特定の要件による検索を行う<BR>
     * 複数指定された場合はAND検索となる
     * @param  int                                                    $offset 取得開始位置
     * @param  int                                                    $limit  取得数上限
     * @param  int                                                    $id     ID
     * @param  string                                                 $lang   言語
     * @param  string                                                 $src    翻訳元
     * @param  string                                                 $result 翻訳結果
     * @param string $author 修正を行ったユーザ名
     * @return キャッシュの内容.ない場合はnullとなる.
     */
    public function search($offset, $limit, $id, $lang, $src, $result, $author)
    {
        $res->rows = $this->createCond($id, $lang, $src, $result, $author)->
                            limit($limit)->
                            offset($offset)->
                            find_array();

        $res->records = $this->createCond($id, $lang, $src, $result, $author)->count();

        return $res;
    }

    /**
     * 翻訳内容の更新を行う
     * @param int    $id     対象のID
     * @param string $result 検索結果
     * @param string $author 修正を行ったユーザ名
     * @param int $updated 更新日時
     */
    public function update($id, $result, $author, $updated)
    {
        $ret = \ORM::for_table('translator_cache')
            ->where_equal('id', $id)
            ->find_one();
        $ret->result = $result;
        $ret->updated = $updated;
        $ret->author =$author;
        $ret->save();
    }
}

```  
  
 **MultilanguageTokyoMetro/src/MyLib/MsTranslator.php**   
以下のコードが、先のモデルを使用して、データベースに情報があればそれを返し、なければMicrosoftTranslatorを使用するようにしています。  
  
```php
<?php
namespace MyLib;

/**
 */
class MsTranslator
{
    private $apiKey;
    private $model;
    private $lang;
    private $cache;
    private $needUpdate;
    private $addedlist;
    const BASE_LANG='ja';

    /**
     * コンストラクタ
     * @param string                        $apiKey apiKey
     * @param \Model\MsTranslatorCacheModel $model  キャッシュ用のモデル
     */
    public function __construct($apiKey, $model, $lang)
    {
        $this->apiKey = $apiKey;
        $this->model = $model;
        if ($lang) {
            $this->lang = $lang;
        } else {
            $this->lang = \MyLib\MsTranslator::BASE_LANG;
        }
        $this->cache = $this->model->getCache($this->lang);
        $this->needUpdate = false;
        $this->addedlist = array();
    }

    public function translator($src)
    {
        if (!$src) {
            return $src;
        }
        if (!is_string($src)) {
            return $src;
        }
        if (\MyLib\MsTranslator::BASE_LANG===$this->lang) {
            return $src;
        }
        if (isset($this->cache[$src])) {
            return $this->cache[$src];
        }
        $ret = $this->doApi($src);
        if ($ret) {
            $this->needUpdate = true;
            $this->cache[$src] = $ret;
            array_push($this->addedlist, $src);

            return $ret;
        } else {
            return $src;
        }
    }

    public function needUpdate()
    {
        return $this->needUpdate;
    }

    public function updateCacheDb()
    {
        if (\MyLib\MsTranslator::BASE_LANG===$this->lang) {
            return;
        }
        if ($this->needUpdate()) {
            //$this->model->deleteCache($this->lang);
            $targets = array();
            foreach ($this->addedlist as $a) {
                $targets += array($a => $this->cache[$a]);
            }
            $this->model->addCache($this->lang, $targets);
            $this->needUpdate = false;
            $this->addedlist = array();
        }
    }

    private function doApi($src)
    {
        $ch = curl_init(
            'https://api.datamarket.azure.com/Bing/MicrosoftTranslator/v1/Translate?Text=%27' .
            urlencode($src).
            '%27&To=%27'.
            $this->lang.'%27'
        );
        curl_setopt($ch, CURLOPT_USERPWD, $this->apiKey.':'.$this->apiKey);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
        curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);

        $result = curl_exec($ch);

        $errno = curl_errno($ch);
        if ($errno === 0) {
            $result = explode('<d:Text m:type="Edm.String">', $result);
            $result = explode('</d:Text>', $result[1]);
            $result = $result[0];

            return $result;
        } else {
            $error = curl_error($ch);

            return null;
        }
    }
}

```  
  
  
これらのコードの実際の使い方は以下のようになります。  
以下のテストコードより抜粋  
https://github.com/mima3/MultilanguageTokyoMetro/blob/master/test/MsTranslatorTest.php  
  
  
```php
$app = new \Slim\Slim();
ORM::configure('sqlite::memory:');
$db = ORM::get_db();

// モデルの初期化
$model = new \Model\MsTranslatorCacheModel($app, $db);

// データベースの構築
$model->setup();

// 翻訳情報追加
$en_inp = array();
$en_inp += array('猫'=>'cat');
$en_inp += array('犬'=>'dog');
$model->addCache('en', $en_inp);

$de_inp = array();
$de_inp += array('猫'=>'Katze');
$model->addCache('de', $de_inp);

// 翻訳コントロールの初期化
// MS_AZURE_KEYはマイクロソフトから手一驚されるAPIキー
$trans = new \MyLib\MsTranslator(MS_AZURE_KEY, $model, 'en');

// 翻訳情報が更新されたか調べる。（この場合は未更新なのでFalse)
echo $trans->needUpdate()

// 登録済みの翻訳をする　・・・　結果はcat
echo $trans->translator('猫')

// 翻訳情報が更新されたか調べる。（この場合は未更新なのでFalse)
echo $trans->needUpdate()

// 未登録の翻訳情報を取得するとMsTranslatorAPIを実行して結果が「Humanになる」
echo $trans->translator('人間')

// 翻訳情報が更新されたか調べる。（この場合は更新されたのでTrue)
echo $trans->needUpdate()

// 新規に更新された翻訳情報をDBに書き込む
$trans->updateCacheDb();
```  
  
みてわかる通りアプリケーション開発者としては、文字を使用するときに下記のような実装をするだけで多言語対応がおこなえるようになります。  
  
```
$trans->translator('人間')
```  
  
## まとめ  
MicrosoftTranslatorを用いて機械翻訳を利用して多言語対応を行うことで、低コストで多言語対応のサイトにとりくむことができるという事例を示しました。開発初期で利用することで大きな効果があげられると期待できます。  
  
なお、注意点として以下のような問題があります。  
  
・文字をバカ正直に翻訳しているだけなので、ローカライズ情報でよく使う以下のような書式の検証が不十分です。  
  
「I have %d pens」 %dは可変の整数  
  
・アラビア語などのRTL対応はアプリケーションが個別におこなわなければなりません。ウェブアプリの場合は、CSSに以下のようなスタイルを追加して適用してやるといいでしょう。  
  
```css
.rtl {
direction: rtl;
/*unicode-bidi:bidi-override;*/
}
```  
  
・多言語対応は文字だけでは不十分です。日付の表現、数値の表現にも気をつけましょう。   
 **Node.js + expressにおける多言語化の考察 一般的な多言語対応における注意事項**   
http://qiita.com/mima_ita/items/d7e48126f326a82af4c7#1-6  
