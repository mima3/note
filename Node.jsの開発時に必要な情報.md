この記事では、Node.jsの開発時にデバッグ～継続的インテグレーションが行えるようにするために必要な情報について記述する。  
  
  
# Node.jsのインストール方法  
  
http://nodejs.org/  
  
ここから、インストーラなりソースなり取得してインストールする。  
Windows,Macはインストーラが存在しており、debianとかはソースから作成する。  
Windowsの場合、VS2008や、VS2012などの複数のVisualStudioが混在している環境だとnpm installが失敗することがある。  
その場合は、環境変数を調整してVS2010が動作するようにしておくこと。  
  
# Node.jsのデバッグ方法  
## node-inspectorによるデバッグ  
Node.jsはnode-inspectorとChromeを用いることでデバッグが可能である。  
  
https://github.com/node-inspector/node-inspector  
  
### node-inspectorのインストール方法  
  
下記のコマンドを実行する。  
  
```
npm install -g node-inspector
```  
  
### node-inspectorによるデバッグ方法  
  
次のような手順となる。  
  
1. デバッグ対象のアプリケーションの起動  
2. node-inspectorの起動  
3. Chromeによってnode-inspectorにアクセスしてデバッグを行う。  
  
#### デバッグ対象のアプリケーションの起動方法  
  
```
>node --debug server.js
debugger listening on port 5858
```  
--debugフラグを使用して対象のアプリケーションを起動すると、デバッグポートが表示される。  
もしデバッグポートを変更したい場合は下記のようにする  
  
```
node --debug=5859 app.js
```  
  
もし、一行目でブレークを賭けたい場合は次の通り  
  
```
node --debug-brk=5859 app.js
```  
  
#### node-inspectorの起動  
デバッグ対象のアプリケーションを起動したらnode-inspectorを起動する。  
これにより、ブラウザ経由でデバッグが可能になる。  
  
```
>node-inspector --web-port 8081 --debug-port 5859
```  
  
--debug-portはデバッグ対象のプロセスのポート  
--web-portはブラウザでデバッグするために使用するポートである。  
  
#### Chromeによるデバッグ  
chromeでnode-inspectorを実行時に支持されたURLにアクセスするとデバッグが可能になる。  
先の例だと、以下のようなURLになる。  
  
http://ホスト名:8081/debug?port=5859  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/3f696825-e623-1007-e2d3-a619d5b869aa.png  
  
## Console.logによるオブジェクトの出力  
Console.logを用いることでオブジェクトをコンソールに出力することができるが、この際、一部の内容しか表示されない。  
  
この場合はutilモジュールを用いることですべての内容を表示させることができる。  
  
```
var util = require('util');
console.log(util.inspect(obj,false,null));
```  
  
  
# Node.jsのメモリー使用状況の調査方法  
  
## メモリ使用状況を調べる  
Node.jsにおいて、どのオブジェクトが、どの処理でメモリを確保したかなどの情報を調べるにはnode-webkit-agentを使用する。  
  
https://github.com/c4milo/node-webkit-agent  
  
(1) node-webkit-agentのインストール  
  
```
npm install webkit-devtools-agent
```  
  
(2)下記の命令を任意のソースに記述。  
  
```
var agent = require('webkit-devtools-agent');
```  
  
(3)アプリケーションを起動  
  
(4)以下のコマンドを実行  
  
```
kill -SIGUSR2 <the process id of your nodejs app>
```  
  
(5)その後、Chromeで以下のURLにアクセスする  
http://c4milo.github.io/node-webkit-agent/26.0.1410.65/inspector.html?host=デバッグ対象のホスト名:9999&page=0  
  
その後は、Profileにてメモリのスナップショットを取得すればよい。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/80fe8278-a9fc-6859-0156-b52aea8ec06d.png  
  
## メモリーの増加を調べる方法  
特定の処理において、メモリーが異常に増加していないか調べる方法について説明する。  
  
Node.jsは世代別のガベージコレクションなので、解放したつもりでもメモリが残っている場合がある。そのときは、任意のタイミングでガベージコレクトを行ったあと、メモリのスナップショットを確認する。  
  
ガベージコレクト後も残っているメモリーは確実に、現在使用しているものとなる。  
  
ガベージコレクションを確実に走らせたい場合は以下のような処理が必要である。  
まず、プログラム中でガベージクレクタを実行したい箇所に次のコードを埋め込む。  
  
```
if(global.gc) {
  global.gc();
}
```  
  
そして、アプリを起動する際に--expose_gcを指定する  
  
```
node --expose_gc server.js
```  
  
# 静的解析  
実際にプログラムを実行しないで、リスクのある記述方法を検知することができる。  
これらの処理はJenkinsなどで定期的に実行するとよい。  
  
## gjslintによる静的解析  
  
gjslintは指定のコードが[Google JavaScript Style Guide](http://google-styleguide.googlecode.com/svn/trunk/javascriptguide.xml "Google JavaScript Style Guide")に違反していないか調べるツールである。  
  
### gjslintのインストール方法  
gjslintはPythonで動作するので、Pythonがインストールされていることが前提になる。  
  
(1)easy_installを使用できるようにする。  
  
```
wget http://peak.telecommunity.com/dist/ez_setup.py
python ez_setup.py
```  
  
(2)gjslintをインストールする  
  
```
easy_install http://closure-linter.googlecode.com/files/closure_linter-latest.tar.gz 
```  
  
### gjslintの使用方法  
  
使用例：  
  
```
gjslint --disable 110,1 -r src
```  
  
--disable コンマ区切りで無視するエラーを指定できる。  
-r ディレクトリを再帰的に操作できる。  
  
### closure-linter-wrapperによるXML出力  
gslintのみでは検査結果をXMLとして保存することはできない。XMLとして出力できないと、jenkinsによる定期ビルドに組み込むことができない。  
これを行うためにはclosure-linter-wrapperが必要である。  
https://github.com/jmendiara/node-closure-linter-wrapper  
  
  
(1)closure-linter-wrapperのインストール  
  
```
npm install -g closure-linter-wrapper
```  
  
(2)次のようなtest.jsファイルを作成する。  
  
```
var argv = process.argv;
var gjslint = require('closure-linter-wrapper').gjslint;
var flagsArray = [
  '--nostrict',
  '--nojsdoc',
  '--disable 14'
];
console.log(argv[2] + 'src/*.js');

gjslint({
    src: [
      argv[2] + 'src/*.js'
    ],
    flags: flagsArray
    ,reporter: {
      name: 'gjslint_xml',
      dest: argv[2] + 'gjslint.xml'
    }
  },
  function (err, result) {
  }
);
```  
  
この例では引数でしたフォルダのsrc以下を解析してgjslint.xmlに出力する。  
あまりに大量のデータを一気にgjslint関数に渡すと正常にXMLが作成されないので注意すること。  
  
### Jenkinsによるgjslintの集計  
Jenkinsによりgjslintを定期的に実行し、その結果を集計することができる。  
  
(1) jslintの結果を集計できるようにViolationsを入手する  
Jenkinsの管理 > プラグインマネージャで利用可能タブから「Violations」を指定してインストールする。  
  
(2) Jenkinsでシェルスクリプトを実行するようにして先に作成したclosure-linter-wrapperを使用するスクリプトを実行する  
  
```
NODE_PATH=/usr/local/lib/node_modules
export NODE_PATH
node ${WORKSPACE}/test.js ${WORKSPACE}/
```  
  
(3)ビルド後の処理のjslintに出力ファイルを指定しておく。  
https://qiita-image-store.s3.amazonaws.com/0/47856/144c1967-04c8-8fd4-9de5-56105fe0043a.png  
  
(4)ビルドを行うと次のようなレポートが作成される。  
https://qiita-image-store.s3.amazonaws.com/0/47856/2b23be68-9e61-32c4-ee6f-c6af0eed976d.png  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/f8eff41e-d861-6ab2-7976-1fef6e3ef5d0.png  
  
## platoによるコードメトリックスの収集  
platoを用いることで、ソースコードの行数や複雑度を計測できる。  
https://github.com/es-analysis/plato  
  
インストール方法：  
  
```
npm install -g plato
```  
  
```
plato -r -d report ./src
```  
  
これによりreportフォルダ以下にHTMLが作成される。  
  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/8a1939a3-fe7a-9f23-28ba-7cb21de4e053.png  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/edff3115-1764-b294-8800-2f383fbe5c52.png  
  
Complexityが高いものがバグを発生しやすいので、テストケースの数や、リファクタリングの目安として監視する。  
これもjenkinsで日々作成するといい。  
  
# テストの自動化  
ここではテストの自動化について記述する。  
テストコードについて、JavaScriptで記述でき、学習コストが低くなる方法について述べている。  
  
## jasmine-nodeによるサーバーサイドの単体テスト  
  
サーバサイドの単体テストはjasmine-nodeで記述するといい。  
https://github.com/mhevery/jasmine-node  
  
--junitreportと--outputfolderオプションを用いることでjunitの形式でXMLを出力できる。  
これによりjenkinsでの集計が可能になる  
  
### jasmine-nodeのXML出力で日本語のファイル名にならない場合  
jasmine-nodeは--junitreportを使用することでXML形式としてテスト結果を出力できる。  
しかし、jasmien-nodeが使用しているjasmine-reportersの不具合で日本語が正常に動作しない。  
  
以下のようなファイルが存在するとする。  
  
```
describe("test\\足し算の確認", function() {
  beforeEach(function() {
    // テスト前処理
  });

  afterEach(function() {
    // テスト後処理
  });
  
  it("足し算が正しい", function() {
    expect(addition(1, 2)).toEqual(3);
  });
});
```  
  
  
この場合、期待の出力結果としては、記号が除去されたファイル名「TEST-test足し算の確認.xml」が作成されることが期待される。  
しかし、現状は「TEST-test.xml」となる。  
  
これを修正するパッチは以下のとおりである。  
  
jasmine-nodeのバージョン：1.14.3  
修正パッチ：  
http://needtec.sakura.ne.jp/release/jasmine.junit_reporter.js.patch  
  
パッチ適用のコマンド例：  
  
```
patch -u ../node_modules/jasmine-node/node_modules/jasmine-reporters/src/jasmine.junit_reporter.js < jasmine.junit_reporter.js.patch
```  
  
## jasmine1.3とjs-test-driverによるブラウザサイドの自動化  
ブラウザサイドのテストは同じテストを異なるブラウザで確認しないといけないので、工夫が必要である。  
  
これはjs-test-driverとjasmine1.3とjasmine-jstd-adapterを組み合わせることで実現可能だ。  
  
[WebブラウザでJavaScriptをテストする「js-test-driver」とQUnit、Jasmineを連携してテストするには (4/4)](http://www.atmarkit.co.jp/ait/articles/1301/21/news017_4.html "WebブラウザでJavaScriptをテストする「js-test-driver」とQUnit、Jasmineを連携してテストするには (4/4)")  
  
jasmineは現在2.0も存在するが、js-test-driverと組み合わせて使うには1.3である必要がある。  
これにより、サーバーサイドと同じテストコードの書き方でクライアントも記述できる。  
  
## seleniumによるUIテストの自動化  
seleniumを用いることで、UIのテストが自動化できる。  
seleniu-webdriverを用いればNode.jsでテストを記述することが可能である。  
  
[seleniumを使ってJavaScriptのみでWebUIを自動操作する](http://needtec.exblog.jp/22769037/ "seleniumを使ってJavaScriptのみでWebUIを自動操作する")  
  
ただし、これは導入コストと維持コストが極めて高いので、ここぞという時だけに絞ること。  
