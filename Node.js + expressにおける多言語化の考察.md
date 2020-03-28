# 目的  
本稿はnode.js+expressにおいて多言語化をどのように行うべきかを考察する。  
  
# サンプル  
http://needtec.sakura.ne.jp/release/node_globalization_sample.zip  
  
このサンプルはログイン画面とユーザ登録画面について、多言語化対応できるように実装している。  
  
# 基本的な考え方  
ユーザに提示する、すべてのメッセージは１つのファイルに記述する。  
たとえば、エラー時に表示するメッセージやボタンに表示するテキストがこれにあたる。  
  
今回はroutes/message.jsonがメッセージ情報を格納するファイルとなる。  
  
**routes/message.json**  
```json:routes/message.json
{
  "message": {
    "error": {
      "NotfoundUser": "ユーザ名(%s)は存在しないかパスワードが間違っています.",
      "DatabaseError": "データベースの接続に失敗しました。<BR>%s",
      "ConstraintError": "制約違反が発生しました。%s",
      "UniqueConstraintError": "項目(%s)には同じ値を重複して登録できません。",
      "FieldError": "入力フィールド：%s にてエラーが発生しました。",
      "DbLockError": "データベースファイルがロックされています。システム管理者に問い合わせてください",
      "NotDirectoryError": "指定のパス(%s)はディレクトリではありません。",
      "RequirementFieldError": "指定の項目(%s)は必須項目です。"
    },"ValidationError": {
      "is": "",
      "not": "",
      "isEmail": "Emailの書式のみ許可されています。",
      "isUrl": "URLの書式のみ許可されています。",
      "isIP": "IPv4またはIPv6の書式のみ許可されています。",
      "isIPv4": "IPv4の書式のみ許可されています。",
      "isIPv6": "IPv6の書式のみ許可されています。",
      "isAlpha": "英字のみ許可されています。",
      "isAlphanumeric": "英数字のみ許可されています。",
      "isNumeric": "数字のみ許可されています。",
      "isInt": "整数のみ許可されています。",
      "isFloat": "浮動小数点のみ許可されています",
      "isDecimal": "Decimalのみ許可されています",
      "isLowercase": "小文字のみ許可されています",
      "isUppercase": "大文字のみ許可されています",
      "notNull": "NULLは許可されていません。",
      "isNull": "NULLのみ許可されています。",
      "notEmpty": "空文字は許可されていません。",
      "equals": "%sのみ許可されています",
      "contains": "",
      "notIn": "",
      "isIn": "",
      "notContains": "",
      "len": "文字数は(%d～%d)の範囲にしてください。",
      "isUUID": "",
      "isDate": "",
      "isAfter": "",
      "isBefore": "",
      "max": "",
      "min": "",
      "isArray": "",
      "isCreditCard": ""
    }
  },
  "view": {
    "login": {
      "title" : "ログイン",
      "linkToRegister": "新規登録",
      "user": "ユーザ名",
      "password": "パスワード",
      "doLogin": "ログイン"
    },
    "register": {
      "title" : "ユーザ登録",
      "user": "ユーザ名",
      "password": "パスワード",
      "passwordConfirm": "確認用パスワード",
      "doRegister": "新規登録",
      "MissmatchPassword": "入力したパスワードが一致しません",
      "Success": "ユーザの登録に成功しました。"
    },
    "project": {
      "title" : "プロジェクトの一覧",
      "projectName": "プロジェクト名",
      "path": "パス",
      "saveBtn": "保存",
      "confirmDelete": "プロジェクト:%sを削除しますか？"
    },
    "index": {
      "logout": "ログアウト",
      "projectSetting": "プロジェクト設定",
      "title": "Jasmine Test Runner",
      "projectName": "プロジェクト名",
      "path": "パス",
      "contextRun": "Jasmine実行"
    }
  },
  "model": {
    "User": {
      "username": "メールアドレス",
      "password": "パスワード"
    },
    "Project": {
      "name": "プロジェクト名"
    },
    "RootPath": {
      "path": "パス",
      "ProjectId": "プロジェクト"
    }
  }
}

```  
  
多言語化対応を行う場合は、このmessage.jsonと同様のJSONファイルを対応言語分用意する。  
そして、設定に応じて使用するjsonファイルを切り替える。  
  
プログラムには、直接文字を記述せず必ずmessage.jsonから文字を取得するようにする。  
このさい、util.formatを使用してすると、以下のように動的に文字を構築てきる。  
  
```js
var name = 'Jack'
var c = 10
util.format('%s は %d の リンゴを持っている', name, c)
//-> Jack は 10 のリンゴを持っている
```  
  
この際、注意することして、翻訳の際は%sや%dの順番が同じになるようにしなければならない。  
  
message.jsonに登録する際は文章単位で登録する。けっしてプログラムで連結してはいけない。  
  
```
英語の場合：
str1 // "I have "
str2 // "pen"
strRet= str1 + str2;
// I have pen. ... OK

日本語の場合：
str1 = "私は持っている";
str2 = "ペン";
strRet = str1 + " " + str2;
// 私は持っているペン ... NG
```  
  
言語によって文法は異なるので単語単位の翻訳ではなく、文章単位の翻訳を行うこと。  
  
# 多言語化の例  
実際の多言語化の例をソースコードと共に例示する。  
  
## サーバーの応答の多言語化  
サーバーの応答に含む文字を多言語化する例を以下に示す。  
  
以下の例では、エラーが発生した場合に「ユーザ名(%s)は存在しないかパスワードが間違っています.」と表示する。  
  
この場合、直接、文字をプログラムに記述していない。common.getMessageにコードを指定し、エラーメッセージを作成している。  
  
  
**router/login.js**  
```js:router/login.js
    var data = {};
    if (result) {
      data = {result: true};
      req.session.user = email;
      data = {
        result: true
      };
    } else {
      data = {
        result: false,
        error: common.getMessage('error',
                                 'NotfoundUser',
                                 [email])
      };
    }
    res.contentType('application/json');
    var json = JSON.stringify(data);
    res.send(json);
```  
  
common.getMessageでおこなっていることはmessage.jsonから指定のキーのデータを取得して引数で指定された文字と併せて文字列として返している。  
  
**routes/common.js**  
```js:routes/common.js
var util = require('util');
var message = require('./message.json');
/**
 * メッセージの取得
 * @param {string} category カテゴリ名
 * @param {string} id ID
 * @param {Array} args 引数の配列
 */
function _getMessage(category, id, args) {
  var msg = message.message[category];
  if (msg) {
    if (msg[id]) {
      var param = [msg[id]];
      param = param.concat(args);
      return util.format.apply(this, param);
    }
  }
  return util.format('category:%s id:%s args:%s' ,category, id, args);
}
```  
  
## Viewにおけるメッセージの構築例  
ボタンのタイトルやラベルの文字を多言語化するには、レンダリングするさいに、ビューで必要なメッセージをすべて渡せばよい  
  
以下の例ではユーザの新規登録画面における例を示している。  
以下のように、message.jsonから必要なメッセージを common.getViewMessageを用いてすべて取得して、テンプレートエンジンに渡している。  
  
**routes/login.js**  
```js:routes/login.js
exports.register = function(req, res){
  res.render('register',
             { message: common.getViewMessage('register') });
};
```  
  
ここで使用しているテンプレートは次のようになる。  
  
```views/register.jade
doctype html
html
  head
    title #{message.title}
    link(rel='stylesheet', href='/stylesheets/style.css')
    link(rel='stylesheet', href='/javascripts/lib/msgbox/msgBoxLight.css')
    script(type='text/javascript', src='/javascripts/lib/jquery-1.11.1.min.js')
    script(type='text/javascript', src='/javascripts/lib/msgbox/jquery.msgBox.js')
    script(type='text/javascript', src='/javascripts/src/ui_util.js')
    script(type='text/javascript', src='/javascripts/src/register.js')
  body
    h2 #{message.title}
    form(action='/login/doRegister' method='POST')#registerForm
      div #{message.user}.
        input(type='text', name='user', placeholder='#{message.user}')
      div #{message.password}.
        input(type='password' name='password', placeholder='#{message.password}')
      div #{message.passwordConfirm}.
        input(type='password' name='passwordConfirm', placeholder='#{message.passwordConfirm}')
      div
        button #{message.doRegister}
    include message
```  
  
InputBoxの表題やボタン名などユーザに表示する文字は、直接記述していない。  
  
以下のように、message.jsonのデータが格納されているオブジェクトを経由して表示している。  
  
```
        button #{message.doRegister}
```  
  
  
## クライアントサイドのJavaScriptでメッセージを使用する  
クライアントサイドのJavaScriptで文字を表示しようする場合はどうするべきか？  
  
先のテンプレートの最期に下記のようにincludeを行っている。  
  
```
    include message
```  
  
これは別のmessage.jadeというテンプレートを埋め込むことを意味している。  
  
```message.jade
div#message(style='display:none')
  - for(var key in message) {
    div(id='#{key}') #{message[key]}
  -}
```  
  
このように記述することで、<div id="message">というタグの中に、必要なメッセージを埋め込む。  
  
これを、以下のようなコードで取得すればよい。  
  
```js
$('#message').find('#' + id).text();
```  
  
もし、%sや%dなどを使用したい場合はutil.formatのコードで必要な部分をクライアントサイドのJavaに移植してやればよい。  
  
**public/javascripts/src/ui_util.js**  
```js:public/javascripts/src/ui_util.js
var formatRegExp = /%[sdj%]/g;
function _format(f) {
  if (!_isString(f)) {
    var objects = [];
    for (var i = 0; i < arguments.length; i++) {
      objects.push(arguments[i]);
    }
    return objects.join(' ');
  }

  var i = 1;
  var args = arguments;
  var len = args.length;
  var str = String(f).replace(formatRegExp, function(x) {
    if (x === '%%') return '%';
    if (i >= len) return x;
    switch (x) {
      case '%s': return String(args[i++]);
      case '%d': return Number(args[i++]);
      case '%j':
        try {
          return JSON.stringify(args[i++]);
        } catch (_) {
          return '[Circular]';
        }
      default:
        return x;
    }
  });
  for (var x = args[i]; i < len; x = args[++i]) {
    if (isNull(x) || !isObject(x)) {
      str += ' ' + x;
    } else {
      str += ' ' + inspect(x);
    }
  }
  return str;
}

function _isObject(arg) {
  return typeof arg === 'object' && arg !== null;
}

function _isNull(arg) {
  return arg === null;
}

function _isString(arg) {
  return typeof arg === 'string';
}

function _isUndefined(arg) {
  return arg === void 0;
} 
```  
  
# 別解  
自前で実装せずにnode用の多言語のライブラリを使用する  
https://github.com/mashpie/i18n-node  
  
# 一般的な多言語対応における注意事項  
多言語化で一般的に気を付けることは以下のとおりである。  
  
## 長さ  
　同じ意味を持つ言葉でも言語によって長さが異なる。そのため、描画を行う際には文字が長くなった時のレイアウトも考慮しなければならない。  
  
## 改行  
ラベルなどのコントロールに複数行の文字を表示する場合があります。 この場合、改行の方法は言語によってことなる。  
  
日本語の場合、単語の途中で改行する事は許されている。  
  
```
天の光はすべ
て敵ですか？
```  
  
しかし英語の場合、単語の途中で改行することは許されていない。  
  
NG  
  
```
Are all the ligh
ts in the heavens our enemy?
```  
  
OK  
  
```
Are all the lights 
in the heavens our enemy?
```  
  
ブラウザを使用するなら適切に描画してくれるが、もし自分で改行を行う場合は注意すること。  
  
## 比較とソート  
地域によってアルファベットの順番は異なり、ソート順と文字の比較に影響を与える。  
たとえばデンマーク言語ではÆをアルファベットのZの後にくる独特な文字として扱う。  
以下のような配列をソートしたとする。  
  
```
[0]: Apple [1]: Æble [2]: Zebra
```  
  
"en-US"の環境では下記のソート順にする必要がある  
  
```
    [0]: Æble [1]: Apple [2]: Zebra
```  
  
"da-DK"の環境では下記のソート順にする必要がある。  
  
```
    [0]: Apple [1]: Zebra [2]: Æble
```  
  
## 複数形  
数字と組み合わせて表示を行う場合、複数を考える必要がある。  
英語には 2 種類の複数形、単数と複数しかないが、スロベニア語には 4 つの複数形があるので単純なifではいけない。  
  
以下のように()でごまかす場合も多々ある。  
  
```
2 photo(s).
```  
  
## 色とアイコン  
色とアイコンも地域化の場合に注意を払う必要がある項目である。  
文化によって色やアイコンの意味合いがことなることがある。  
  
## 文字の方向  
英語は左から右に読みますが、アラビア語は右から左に読みます。  
右から左に読む言語は以下の通りになります。  
  
 **Questions & Answers: Which languages are written right-to-left (RTL)?**   
http://www.i18nguy.com/temp/rtl.html  
  
## その他  
・タイムゾーン  
・日付の表記方法　yyyy-mm-dd か dd-mm-yyyy ...  
・金額の表記方法  
・数値の表記方法　1,053.3　か　1.050,3　か 1’050.3 ...  
  
  
# 参考  
 __国際対応アプリケーションの開発__   
http://msdn.microsoft.com/ja-jp/library/aa719955(VS.71).aspx  
  
 __ウェブサイトのローカライズで気をつけたい6つのポイント__   
http://builder.japan.zdnet.com/sp/web-design-localize-2008/  
