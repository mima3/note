# 概要  
このドキュメントは、TestLink1.7系からTestLink1.9にバージョンアップした際の違いについて記述する。  
  
一般的なTestLinkの説明については下記を参照  
  
 __脱Excel！ TestLinkでアジャイルにテストをする (1/6)__   
http://www.atmarkit.co.jp/ait/articles/0910/23/news110.html  
  
# WindowのXAMPPにインストールした場合の問題  
Window7のXAMPP1.8.3にTestLink1.9.10を導入した場合、下記のエラーが発生して動作しない。  
  
```
Fatal error: Uncaught exception 'SmartyCompilerException' with message 'Syntax Error in template &quot;C:\xampp183\htdocs\testlink\gui\templates\login.tpl&quot; on line 12 &quot;&amp;lt;script language=&amp;quot;JavaScript&amp;quot; src=&amp;quot;{$basehref}gui/niftycube/niftycube.js&amp;quot; type=&amp;quot;text/javascript&amp;quot;&amp;gt;&amp;lt;/script&amp;gt;&quot; unknown tag &quot;private_print_expression&quot;' in C:\xampp183\htdocs\testlink\third_party\smarty3\libs\sysplugins\smarty_internal_templatecompilerbase.php:665 Stack trace: #0 C:\xampp183\htdocs\testlink\third_party\smarty3\libs\sysplugins\smarty_internal_templatecompilerbase.php(451): Smarty_Internal_TemplateCompilerBase->trigger_template_error('unknown tag "pr...', 12) #1 C:\xampp183\htdocs\testlink\third_party\smarty3\libs\sysplugins\smarty_internal_templateparser.php(2353): Smarty_Internal_TemplateCompilerBase->compileTag('private_print_e...', Array, Array) #2 C:\xampp183\htdocs\testlink\third_party\smarty3\libs\sysplugins\smarty_internal_templatepa in C:\xampp183\htdocs\testlink\third_party\smarty3\libs\sysplugins\smarty_internal_templatecompilerbase.php on line 665
```  
  
この問題に関しては、下記のフォルダを最新のSmarty3に置き換えれば回避できる。  
  
testlink\third_party\smarty3  
  
http://www.smarty.net/download  
Smarty 3.1.18(安定板）をダウンロードする。  
  
なお、Smarty3のエラーはWindowsまたはXAMMPとの組み合わせの問題だと思われる。  
1.9.9でDebianを対象に環境を構築した時はこの問題は発生しなかった。  
  
  
また、XAMMPの場合PHPのタイムゾーンが異なっているので適切に修正すること。  
php.ini ファイルの下記修正  
  
デフォルトの設定  
  
```
date.timezone = Europe/Berlin
```  
  
東京の場合  
  
```
date.timezone = Asia/Tokyo
```  
  
# 1.7⇒1.9の間に追加された機能  
  
・要件仕様の作成が可能になっている。要件とテスト仕様書の関連付がおこなえる。  
  
・Platform Management よりプラットフォームを追加できる。これにより、同じテストケースを異なるプラットフォームでテストすることができる。  
今回のテスト計画ではWindowsだけテストする。次のテスト計画ではすべてのOSをテストするといった使い方ができる。  
  
・TracやRedmineとの連携は「Issue Tracker Management」で行うようになっている。各テストプロジェクトごとに設定できる。  
TracやRedmineのサーバが動作していないときにテストの実行を行おうとすると固まるので、注意が必要。エラーメッセージも表示されない。  
  
・CSVやExcelからのインポートができない。XMLからのみである。かってCSVからのインポートをサポートしたTestLinkCnvMacroはすでに存在しない。  
  
・XML-RPCならびにRESTAPIで外部アプリから操作が可能である。  
  
・上記にともない各テストケースに実行タイプが追加され、「手動」または「自動」の選択を行える。  
  
# Excelからのインポートを考える  
  
http://needtec.sakura.ne.jp/release/testlink.xlsm  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/c045b1aa-c545-8fae-9c5f-d0751b5f6313.png  
  
  
このExcelは入力した項目をTestLink用のXMLに変換してファイルとして出力する。  
ステップと期待値はALT＋ENTERでセル内で改行すると、それを１ステップとしてかぞえる。  
中項目、小項目は省略でき、その場合は大項目の下にテストケースが作成される。  
XMLへの変換はVBAのMSXMLで行っている。  
  
# Testlinkを利用した自動テストについて考える  
jUnitやnUnitといったテストコードを記述し、かつ、その結果が適切に集計、出力されるものはTestlinkで管理するべきではないと考える。  
これは、テストケースの記述がテストコードとTestlinkの両方に書かねばならなくなり、二重管理になるからだ。  
また、xUnitで行うテスト項目をTestLinkでテストケースを記述するのは粒度の問題で適切ではないと思われる。  
  
TestLinkを利用した自動テストは結合フェーズ以降で行うとよい。  
これならば、テスト結果の集計処理などをTestLinkにまかせることができるので意味がでてくる。  
具体的には以下のようなテストが考えられる  
　・画面をSeleniumやUWSCで動かしてその結果をTestLinkに登録する。  
　・バッチファイルからテスト対象プロセスを起動して、その結果作成されたファイルの是非をTestLinkに登録する。  
  
  
## REST APIの使用方法  
ここではRESTAPIを使用してテストの実行結果を記録する方法を紹介する。  
結論からいうとXML-RPCを使用したほうがいい。  
  
１．APIキーを作成し取得する。  
https://qiita-image-store.s3.amazonaws.com/0/47856/e8298f3e-b24f-b817-2683-78b01d39a2fe.png  
  
2.Pythonで以下のようなコードを作成する。  
このコードはテストの実行結果を記録するものである。  
  
```py
# -*- coding:utf-8 -*-
import urllib2
import json

# 個人設定画面に表示されているAPIキー
api_key = 'e3c82f414672b5c23d6688ea03ba8b36'


# テストの実行
data = {
  # testlink.executions に以下のデータを記録する
  "testPlanID": 221, # データベースに登録されているID　
  "buildID": 1,      # データベースに登録されているID
  "platformID": 0,   # データベースに登録されているID
  "testCaseExternalID": "test2-4", # テストケースを編集する際に表示されるID
  "notes": "備考の表示",  # 備考
  "statusCode": "f",  # p:成功 f:失敗 b:ブロック
  "executionType": 2  # 1:手動 2:自動
  
}
url = 'http://localhost/testlink/lib/api/rest/v1/executions'
req = urllib2.Request(url, json.dumps(data))
req.add_header('Content-Type', 'application/json')
req.add_header('Php-Auth-User',  api_key) #PHP_AUTH_USERとしてはいけない。
req.get_method = lambda: 'POST'
response = urllib2.urlopen(req)
ret = response.read()
print 'Response:', ret
```  
  
このRESTAPIには、いくつか注意点がある。  
・add_headerで追加する際に「PHP_AUTH_USER」としてはいけない。  
クライアントのリクエストに「_」が混ざると正常にヘッダが認識されない。  
なお、「-」はサーバー側で確認すると「_」に変換されている。  
  
・testlink.executionsテーブルに記録する場合、データベースで使用しているIDが必要になる。  
このIDは画面からも取得できないし、他のRESTAPIでも取得できない。  
データベースを開いて確認するしかない。  
  
・ロジックを見る限り、テスト計画の取得などの機能が実装されていない。  
  
・ドキュメントはなさそうなのでtlRestApi.class.phpを自分で解析する必要がある。  
  
・executionsのパラメータは下記の通り  
  
|プロパティ|説明|  
|:---------|:---|  
|testPlanID|テスト計画のデータベースに登録されている内部のID|  
|buildID|リリース/ビルドのデータベースに登録されているID|  
|platformID|プラットフォームのデータベースに登録されているID|  
|testCaseExternalID|テストケースの画面表示されるID|  
|notes|テスト結果の備考|  
|statusCode|テスト結果<BR>p:成功<BR>f:失敗<BR>b:ブロック|  
|executionType|実行タイプ<BR>1:手動 2:自動|  
  
  
  
ここまでRESTAPIでTestLInkを操作した感想だが、「DBを直接開いて内部のIDを調べる」、「操作の仕様はソースコード。そして全部の機能はない」とかいう状況だと、他人に作業をふれないので、今しばらく避けたほうがいい。  
  
  
  
# 参考  
TestLink の REST API を触ってみた   
http://qiita.com/bamchoh/items/430e07b084ffbd1c0b36  
