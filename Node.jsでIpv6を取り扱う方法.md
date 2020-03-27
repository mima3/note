ここではNode.jsにおいてIpv6をどのように取り扱うか説明する。  
  
# ExpressのサーバーでIPv6を受け付けるようにする。  
既定の設定でNode.js+Expressでサーバーを立ち上げた場合、下記のようにipv6のアドレスで接続しようとすると失敗する。  
  
```
http://[fe80::20c:29ff:feb8:6dd1]:3000/
```  
  
この回避策としてIPを明示する方法が提示されている。  
http://code.danyork.com/2011/01/21/how-to-use-node-js-with-ipv6/  
  
```js
var http = require('http');
 
var handler = function (request, response) {
   response.writeHead(200, {"Content-Type":"text/plain"});
   response.end ("Hello World!n");
   console.log("Got a connection");
};
 
var server6= http.createServer();
server6.addListener("request",handler);
server6.listen(80,"2001:db8:1111:2222:3333::51");
 
var server= http.createServer();
server.addListener("request",handler);
server.listen(80,"192.0.2.23");
 
console.log("Server running on localhost at port 80");
```  
  
この方法はイーサーネットを複数保持している場合は動作しない。  
たとえば、上記でlistenしているアドレスのほかに2001::52のイーサーネットを保持していた場合、2001::52からのアクセスがみとめられない。  
  
保持するIpv6すべてでlistenするには以下のようにする。  
  
  
```js
    var server = app.listen(app.get('port'), '::0', function() {
      debug('Express server listening on port ' + server.address().port);
    });
```  
  
Ipv6形式で記述してあるが、ipv4のアドレスも受け付ける。  
  
# IPv6の操作  
Node.jsでIpv6の有効チェックや数字化、範囲チェックを行うには以下のライブラリを使用するとよい。  
  
https://github.com/beaugunderson/javascript-ipv6  
  
IPv6を数値に変換した場合、JavaScriptの仕様上、double型と扱われ、精度が落ちる。  
このライブラリではbigIntegerというライブラリを用い、それを回避している。  
もし、IPの範囲の検査で大小を比較する場合は、bigIntegerが提供しているcompairToを用いて行う。  
まちがっても自分ですべての配列の大小を比較する車輪の再開発みたいなことはしてはいけない。  
  
  
以下にその使用例を示す。  
  
```js
var v6 = require('ipv6').v6;
var address1 = new v6.Address('2001:0:ce49:7601:e866:efff:62c3:fffe');
var address2 = new v6.Address('2001:0:ce49:7601:e866:efff:61c3:0002');

// 値がipv6かチェックする
console.log(address1.isValid());

var ngAddress = new v6.Address('1.2.3.4');
console.log(ngAddress.isValid());

var ngAddress = new v6.Address('test.co.jp');
console.log(ngAddress.isValid());

// アドレスの比較をする場合はbigIntegerに変換して、compareToで比較すること
var bi1 = address1.bigInteger();
var bi2 = address2.bigInteger();

// thisがでかければ+
console.log(bi1.compareTo(bi2));
// ひとしければ 0
console.log(bi1.compareTo(bi1));
// 引数のほうが大きければ-
console.log(bi2.compareTo(bi1));

// サブネットマスクを指定して、開始アドレス、終了アドレスも取得できる。
address1.subnetMask = 64;
console.log(address1.startAddress());
console.log(address2.endAddress());


// 省略形で表示も可能
address1 = new v6.Address('2001:0:0:0:0:0:0:1');
// 2001::1 と省略形で
console.log(address1.correctForm());

```  
