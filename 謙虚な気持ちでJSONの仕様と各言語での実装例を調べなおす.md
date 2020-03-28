# JSONの仕様  
JavaScript Object NotationはJavaScriptから派生したものですが、2019年現在、多くのプログラミング言語にてJSON形式のデータを生成および解析するためのコードが含まれています。  
  
仕様改定の歴史としては以下のようになっています。  
  
|時系列|仕様||  
|:--|:---|:--|  
|2006年7月|[RFC4627](https://www.ietf.org/rfc/rfc4627.txt)||  
|2013年3月|[RFC7158](https://tools.ietf.org/html/rfc7158)|[RFC4627](https://www.ietf.org/rfc/rfc4627.txt)ではJSONの文章はobjectまたはarrayを必ず含む必要がありましたが、その制限が解除されました。|  
|2013年10月|ECMA-404 1st edition||  
|2014年3月|[RFC7159](https://tools.ietf.org/html/rfc7159)||  
|2017年12月14日|[RFC8259](https://tools.ietf.org/html/rfc8259)およびIETF STD 90および[ECMA-404 2nd edition](https://www.ecma-international.org/publications/files/ECMA-ST/ECMA-404.pdf)|[RFC8259](https://tools.ietf.org/html/rfc8259)にて文字コードはBOMなしのUTF-8でエンコードすることが必須となりました。|  
  
## JSON values  
JSONの値はobject, array, number, string, true, false, null となります。  
![image.png](/image/2be41691-e325-69ca-55a9-2b9aa02a0961.png)  
  
## Object  
objectは0個以上の名前/値のペアを囲む、一対のトークンとして表せられます。名前はstringになります。名前の後に「:」とトークンが続き、名前と値を分けます。単一の「,」は値を次の名前から分離します。  
名前と値のペアの順序に重要性はありませんし、**名前文字列が一意であることを要求するものではありませんし、複数あった場合の挙動は定義されていません。**  
![image.png](/image/0d6d06c7-1f19-189b-92ce-d0203f7b58cb.png)  
  
## Array  
arrayは0個以上の値を囲む角括弧トークンです。値は「,」で区切ります。  
![image.png](/image/e6d4393d-df38-5547-0a75-9ee0868758fa.png)  
  
## Number  
numberは余分の先頭0が存在しない10進数です。  
マイナス記号、小数点、eまたはEが付与される場合があります。  
数字として表すことのできないNaNやInfinityは許可されません。  
  
![image.png](/image/0929f143-e7f1-cc98-0a1d-5a90e0ec8008.png)  
  
10 → Numberとみなす  
-10 → Numberとみなす  
10.5 → Numberとみなす  
1e+3 → Numberとみなす  
1E-3 → Numberとみなす  
-0   → Numberとみなす  
0010 → Numberとみなさない  
0x10 → Numberとみなさない  
NaN  →　Numberとみなさない  
  
## String  
stringは「\」によるエスケープシーケンス記法を含む、「"」でくくった文字列です。  
  
![image.png](/image/a3b902c2-c16c-db71-ac62-890ef31db6cc.png)  
  
**\u4 hexadecimal digits**の例は以下のようになります。  
・"\u3041"→"ぁ"となります。  
・"\u002F"と"\u002f"は両方とも有効  
  
# 実装例  
ここでは以下のプログラミング言語で実際にJSONを操作してみます。  
・JavaScript  
・Python  
・C#  
・PowerShell  
・Excel VBA  
  
## 操作対象のJSONの例  
  
**いろいろなValueを読み込むJSON**  
  
```json:test001.json
{
  "null" : null,
  "num1" : 12345,
  "num1" : 99999,
  "num2" : 123.45,
  "num3" : -123.45,
  "num4" : 1e+3,
  "num5" : 1E-3,
  "bool1" : true,
  "bool2" : false,
  "str1" : "abあいうえおcdef",
  "str2" : "str2:\n\r\\\t\/abc\b\u0030\u3041",
  "obj1" : {
    "a" : 125,
    "b" : {
       "c" : {
          "d1": {
             "v" : 1234
          },
          "d2" : 12345
       }
    }
  },
  "ary1" : [
    1,
    2,
    "test",
    {
      "a": 5.3
    }
  ],
  "ary2" : [
    1,
    2,
    3
  ]
}

```  
  
**大量データのJSON**  
  
```json:test003.json
[
  { "num1" : 1, "num2" : 2 },
  { "num1" : 2, "num2" : 2 },
  { "num1" : 3, "num2" : 2 },
  { "num1" : 4, "num2" : 2 },
  { "num1" : 5, "num2" : 2 },
  // 数百メガバイトになるまで繰り返し
]
```  
  
## JavaScript  
環境:  
Windows10  
node.js v10.16.0  
  
### 標準のJSONオブジェクト  
標準のJSONオブジェクトのparseとstringifyでJSONの操作が行えます。  
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON  
  
```javascript
var fs = require("fs");
var contents = fs.readFileSync("C:\\share\\jsontest\\test001.json");
var jsonData = JSON.parse(contents);
console.log(jsonData['null']);
console.log(jsonData['num1']);
console.log(jsonData['num2']);
console.log(jsonData['num3']);
console.log(jsonData['num4']);
console.log(jsonData['num5']);
console.log(jsonData['str1']);
console.log(jsonData['str2']);
console.log(jsonData['obj1']);
console.log(jsonData['obj1']['b']['c']);
console.log(jsonData['ary1']);
console.log(jsonData['ary1'][0]);

// https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify
console.log(JSON.stringify(jsonData));

```  
  
出力結果：  
  
```text
null
99999
123.45
-123.45
1000
0.001
abあいうえおcdef
str2:
\       /ab0ぁ
{ a: 125, b: { c: { d1: [Object], d2: 12345 } } }
{ d1: { v: 1234 }, d2: 12345 }
[ 1, 2, 'test', { a: 5.3 } ]
1
{"null":null,"num1":99999,"num2":123.45,"num3":-123.45,"num4":1000,"num5":0.001,"bool1":true,"bool2":false,"str1":"ab あいうえおcdef","str2":"str2:\n\r\\\t/abc\b0ぁ","obj1":{"a":125,"b":{"c":{"d1":{"v":1234},"d2":12345}}},"ary1":[1,2,"test",{"a":5.3}],"ary2":[1,2,3]}
```  
  
JSON.parseでJSONからJavaScriptのオブジェクトに、JSON.stringify()でJavaScriptのオブジェクトからJSONに変換できていることが確認できます。  
また、num1は重複したデータですが,エラーにはならず、後優先で値が出力されています。  
  
#### JSONの各値の変換処理  
[JSON.parse](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse)のドキュメントによると引数にreviver が設定できることがわかります。  
ここには関数を指定でき，JSONから取得した値を変換することが可能です。  
下記の例は数値型をDecimal型に変換するサンプルとなります。  
  
  
```text:事前準備decimalのインストール

npm install decimal
```  
  
  
  
```javascript
var fs = require("fs");
// https://www.npmjs.com/package/decimal
var Decimal = require('decimal');
var contents = fs.readFileSync("C:\\share\\jsontest\\test001.json");
// https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse
var jsonData = JSON.parse(contents, function (key, value) {
    //console.log(key, JSON.stringify(value));
    if (typeof value === 'number') {
      return Decimal(value);
    }
    return value;
  }
);

console.log('-----------------------------------------');

console.log(jsonData['num1'].constructor.name);
console.log(jsonData['num1'].toString());

console.log(jsonData['num2'].constructor.name);
console.log(jsonData['num2'].toString());

console.log(jsonData['obj1']['a'].constructor.name);
console.log(jsonData['obj1']['a'].toString());


```  
  
出力結果  
  
```text
-----------------------------------------
Decimal
99999
Decimal
123.45
Decimal
125

```  
  
出力結果より、JSON中のNumber型が全てDecimal型にJavaScriptのオブジェクトとして作成されたことが確認できます。  
  
### JSONStream  
大きなサイズのJSONファイルを操作する場合、jsonライブラリではメモリを大量に消費し解析に失敗する可能性があります。  
この場合は、[JSONStream](https://github.com/dominictarr/JSONStream)を使用する必要があります。JSONStreamを使用することで、JSONのデータを一度に全てメモリに展開することなく解析することが可能になります。  
  
  
```text:インストール

npm install JSONStream
```  
  
```javascript
// https://stackoverflow.com/questions/15121584/how-to-parse-a-large-newline-delimited-json-file-by-jsonstream-module-in-node-j
var fs = require('fs'),
    JSONStream = require('JSONStream');

var stream = fs.createReadStream("C:\\share\\jsontest\\test003.json", {encoding: 'utf8'}),
    parser = JSONStream.parse('.');

stream.pipe(parser);

var cnt = 0
parser.on('data', function (obj) {
  // console.log(obj); 
  if (cnt === 0) {
    console.log(obj);
  }
  cnt = cnt + 1;
});

parser.on('end', function () {
  console.log(cnt);
  console.log('end');
});
```  
  
ファイルストリームを作成してそれをJSONStreamのparserに渡します。  
この際、オブジェクトを取得するたびに「data」イベントが発火し、すべてのストリームが完了したら「end」イベントが発火します。  
このサンプルでは「data」イベント中で以下の処理をしています。  
　・最初に取得したオブジェクトの内容を表示  
　・dataイベントが発火した数を数える  
  
また「end」イベントでは「data」イベントが発火した数を表示しています。  
  
以下は出力結果になります  
  
```text
C:\dev\node>node jsontest3.js
{ num1: 1, num2: 2 }
6636801
end
```  
  
詳しい使い方については下記を参照してください。  
https://github.com/dominictarr/JSONStream  
  
## Python  
環境：  
Windows10  
Python 3.7.4  
  
### jsonライブラリ  
Pythonの標準ライブラリの[jsonライブラリ](https://docs.python.org/ja/3/library/json.html)を使用することでJSONの操作を行うことが可能です。  
  
以下のサンプルは「操作対象のJSONの例」を解析するためのプログラムです。  
  
```python
# coding:utf-8
import json


f = open('C:\\share\\jsontest\\test001.json', 'r', encoding="utf-8")

jsonData = json.load(f)
print ('----------------------------------------------------')
print(json.dumps(jsonData, sort_keys = True, indent = 4))
print ('----------------------------------------------------')
print (jsonData['null'])
print (jsonData['num1'])
print (type(jsonData['num1']))
print (jsonData['num2'])
print (type(jsonData['num2']))
print (jsonData['str1'])
print (jsonData['str2'])
print (type(jsonData['obj1']))
print (jsonData['obj1'])
print (jsonData['obj1']['a'])
print (type(jsonData['ary1']))
print (jsonData['ary1'])
print (jsonData['ary1'][0])

f.close()
```  
  
出力結果は以下のようになります。  
  
```text
C:\dev\python3>python jsontest.py
----------------------------------------------------
{
    "ary1": [
        1,
        2,
        "test",
        {
            "a": 5.3
        }
    ],
    "ary2": [
        1,
        2,
        3
    ],
    "bool1": true,
    "bool2": false,
    "null": null,
    "num1": 99999,
    "num2": 123.45,
    "num3": -123.45,
    "num4": 1000.0,
    "num5": 0.001,
    "obj1": {
        "a": 125,
        "b": {
            "c": {
                "d1": {
                    "v": 1234
                },
                "d2": 12345
            }
        }
    },
    "str1": "ab\u3042\u3044\u3046\u3048\u304acdef",
    "str2": "str2:\n\r\\\t/abc\b0\u3041"
}
----------------------------------------------------
None
99999
<class 'int'>
123.45
<class 'float'>
abあいうえおcdef
str2:
\       /ab0ぁ
<class 'dict'>
{'a': 125, 'b': {'c': {'d1': {'v': 1234}, 'd2': 12345}}}
125
<class 'list'>
[1, 2, 'test', {'a': 5.3}]
1
```  
  
json.load()でJSONファイルを展開してJSON中のvalueをPythonのデータに下記の表に従って変換します。  
  
|JSON|Python|  
|:-----|:-----|  
|object|dict|  
|array|list|  
|string|str|  
|number|int または 浮動小数点|  
|true|True|  
|false|False|  
|null|None|  
  
出力結果を確認すると上記の表にしたがって型が変換されていることが確認できると思います。  
また、元データのJSONには「num1」という名称は2つ存在していますが、出力結果を見ると、後の方のデータが使用されていることが確認できます。  
  
また、json.dumps()はPythonのデータをJSONに変換しています。  
  
[jsonライブラリ](https://docs.python.org/ja/3/library/json.html)を確認するとjson.loadにはいくつか興味深いパラメータがあります。次節ではそれらを確認してみましょう。  
  
#### 数値の型に対するコールバック関数  
json.loadにはparse_floatとparse_intの二つの引数があります。これらの引数にはコールバック関数が設定できます。  
  
以下は、コールバック関数を用いてJSONのnumber型をdecimalに変換した例になります。  
  
```python
# coding:utf-8
import json
import decimal

def callbackInt(d):
  return decimal.Decimal(d)

f = open('C:\\share\\jsontest\\test001.json', 'r', encoding="utf-8")

jsonData = json.load(f, parse_float = decimal.Decimal, parse_int = callbackInt)
print ('--------------------------------------------')
print (jsonData['num1'])
print (type(jsonData['num1']))
print (jsonData['num2'])
print (type(jsonData['num2']))
print (jsonData['num3'])
print (type(jsonData['num3']))

f.close()
```  
  
**出力結果**  
  
```text
--------------------------------------------
99999
<class 'decimal.Decimal'>
123.45
<class 'decimal.Decimal'>
-123.45
<class 'decimal.Decimal'>
```  
  
#### オブジェクトの型に対するコールバック関数  
json.loadにはobject_hookというコールバック関数があります。これはJSON中にobjectを検知するたびに実行される関数です。  
数値型に対するコールバック関数と同様に、これを用いてオブジェクトを任意の型に変換することも可能です。  
  
以下の例ではオブジェクト型を検知するたびに、検知した値を出力しています。  
  
```python
# coding:utf-8
import json

def hook_func(dct):
  print (dct)
  return dct

f = open('C:\\share\\jsontest\\test001.json', 'r', encoding="utf-8")

jsonData = json.load(f, object_hook=hook_func)
print ('----------------------------------------------------')

f.close()
```  
  
**出力結果**  
  
```text
C:\dev\python3>python jsontest_1.py
{'v': 1234}
{'d1': {'v': 1234}, 'd2': 12345}
{'c': {'d1': {'v': 1234}, 'd2': 12345}}
{'a': 125, 'b': {'c': {'d1': {'v': 1234}, 'd2': 12345}}}
{'a': 5.3}
{'null': None, 'num1': 99999, 'num2': 123.45, 'num3': -123.45, 'num4': 1000.0, 'num5': 0.001, 'bool1': True, 'bool2': False, 'str1': 'abあいうえおcdef', 'str2': 'str2:\n\r\\\t/abc\x080ぁ', 'obj1': {'a': 125, 'b': {'c': {'d1': {'v': 1234}, 'd2': 12345}}}, 'ary1': [1, 2, 'test', {'a': 5.3}], 'ary2': [1, 2, 3]}
----------------------------------------------------

C:\dev\python3>
```  
  
出力結果を見てわかる通り、基本的に上から見ていきますが、オブジェクトの中にオブジェクトがある場合は、もっとも深い箇所から検知されていきます。  
任意のオブジェクトに変換する場合は、コールバック関数の引数であるdictの全てのキーをみて判断するか、任意の項目に変換すべき型の情報を記載しておく必要があるでしょう。  
  
#### オブジェクトや配列を含まないデータの場合  
最近のJSONの仕様ではオブジェクトや配列を含まない場合もJSONをパースできます。  
これは以下のようなコードで検証できます。  
  
```python
import json
json.loads('12345')
# 12345
```  
  
[jsonライブラリ](https://docs.python.org/ja/3/library/json.html)においては、この仕様を満たしています。  
  
#### NaNやInfinityなどの数値じゃない値を含む場合  
  
```python
import json
n = json.loads('NaN')
print(n)
# nan
json.dumps(n)
# 'NaN'
```  
  
NaNやInfinityを含む場合、デフォルトload/loadsの挙動では**nan**や**inf**に変換されます。  
また、nanやinfを含むPythonの値をdump/dumpsでJSON化した場合、NaNやInfinityに変換されます。  
  
もし、仕様と合わせてJSON中のNaNやInfinityの値をエラーとする場合は以下のようにします。  
  
```python
def errorfunc(d):
   print(d)
   raise

n = json.loads('NaN', parse_constant=errorfunc)
```  
  
これを実行すると、NaNなどを検知した場合にerrorfuncが実行されて例外が発生します。  
また、逆にPythonのデータをJSON化する場合にNaNやInfinityを認めない場合は以下のような実装になります。  
  
  
```python
n = json.loads('NaN')
json.dumps(n, allow_nan=False)
```  
  
この場合、dumps実行時にnanやinfを検出すると「ValueError: Out of range float values are not JSON compliant」が発生します。  
  
### ijsonライブラリ  
大きなサイズのJSONファイルを操作する場合、jsonライブラリではメモリを大量に消費し解析に失敗する可能性があります。  
この場合は、[ijson](https://pypi.org/project/ijson/)を使用します。  
  
ijsonはイテレータのインターフェイスを経由してJSONを解析することにより、すべてのJSONデータをメモリに展開する必要がなくなります。  
以下のサンプルコードは大量データのJSONを解析している例です。  
  
```python
# coding:utf-8
import ijson

def test1():
  # 数百メガのファイルを開く
  f = open('C:\\share\\jsontest\\test003.json', 'r', encoding="utf-8")
  items = ijson.items(f, 'item')
  l = 0
  for i in items:
    if l == 0:
      print (i)  # 最初のオブジェクトのみ表示
    l = l + 1
  print (l)
  f.close()


if __name__ == "__main__":
  test1()
```  
  
ijson.itemsにfile object と prefixを与えて抽出を行います。prefixを設定することで、特定の要素のみを抽出対象にすることができます。今回はarrayの各要素を取得したいので「item」を指定してます。  
  
このファイルの実行結果としては以下のように、最初のオブジェクトと、オブジェクトの総数を出力します。  
  
```text
C:\dev\python3>python jsontest3_3.py
{'num1': 1, 'num2': 2}
6636801
```  
  
**ijson**  
https://pypi.org/project/ijson/  
  
**ijson - Github**  
https://github.com/isagalaev/ijson  
  
## C＃  
環境:  
Windows10  
VisualStudio2019  
.NET 4.7.2  
  
### Json.NET  
NewtonSoftが提供するJson.NETを持ちいることでJSONに対する操作を行います。  
https://www.newtonsoft.com/json  
  
ASP.NETで開発した人は、おそらく良く見るライブラリかと思います。  
  
NuGetで以下のパッケージをインストールしてください。  
![image.png](/image/bcbf2326-25f4-19fc-da0d-8f5172df0cad.png)  
  
#### 型を指定しない使用方法  
JsonConvert.DeserializeObjectに型を指定しないでJSONデータを読み込むことができます。これにより、あらかじめJSONの形に合わせたクラスを用意する必要がなくなります。  
  
```csharp
        static void Test1()
        {
            var contents = File.ReadAllText(@"c:\share\jsontest\test001.json");
            dynamic json = JsonConvert.DeserializeObject(contents);
            Console.WriteLine(json.num1.Type);
            Console.WriteLine(json.num1.Value);

            Console.WriteLine(json.num2.Type);
            Console.WriteLine(json.num2.Value);

            Console.WriteLine(json.num3.Type);
            Console.WriteLine(json.num3.Value);

            Console.WriteLine(json.num4.Type);
            Console.WriteLine(json.num4.Value);

            Console.WriteLine(json.num5.Type);
            Console.WriteLine(json.num5.Value);

            Console.WriteLine(json["null"].Type);
            Console.WriteLine(json["null"].Value == null);

            Console.WriteLine(json.bool1.Type);
            Console.WriteLine(json.bool1.Value);

            Console.WriteLine(json.bool2.Type);
            Console.WriteLine(json.bool2.Value);

            Console.WriteLine(json.str1.Type);
            Console.WriteLine(json.str1.Value);

            Console.WriteLine(json.str2.Type);
            Console.WriteLine(json.str2.Value);

            Console.WriteLine(json.obj1.a.Type);
            Console.WriteLine(json.obj1.a.Value);

            Console.WriteLine(json.ary1.Count);
            Console.WriteLine(json.ary1[0].Type);
            Console.WriteLine(json.ary1[0].Value);

        }
```  
  
出力結果は以下のようになります。  
  
```text
Integer
99999
Float
123.45
Float
-123.45
Float
1000
Float
0.001
Null
True
Boolean
True
Boolean
False
String
abあいうえおcdef
String
str2:
\       /ab0ぁ
Integer
125
4
Integer
1
```  
  
重複している値num1が、後優先で取得されていることが確認できます。  
  
#### 型を指定する方法  
JsonConvert.DeserializeObjectに型を指定して呼び出すことで、JSONの解析結果を事前に用意したクラスに割り当てることも可能です。  
  
```json:使用するJSONデータ
[
  {"name":"Joe", "lv" : 10},
  {"name":"Jack", "lv" : 15},
  {"name":"Sara", "lv" : 13},
  {"name":"Ben", "lv" : 12}
]
```  
  
```csharp
       class Member {
            public string name { get; set; }
            public decimal lv { get; set; }

        }

        static void Test2()
        {
            var contents = File.ReadAllText(@"c:\share\jsontest\test004.json");
            List<Member> json = JsonConvert.DeserializeObject<List<Member>>(contents);
            foreach (var m in json)
            {
                Console.WriteLine(m.name );
                Console.WriteLine(m.lv);
            }

            Console.WriteLine("------------------------------------------");
            Console.WriteLine(JsonConvert.SerializeObject(json, Formatting.Indented));

        }
```  
  
このサンプルではJSONをList\<Member\>に明示的に変換することを行っています。  
また、 List\<Member\>をJsonConvert.SerializeObjectを使用してJSONに戻しています。  
  
  
出力結果：  
  
```text
Joe
10
Jack
15
Sara
13
Ben
12
------------------------------------------
[
  {
    "name": "Joe",
    "lv": 10.0
  },
  {
    "name": "Jack",
    "lv": 15.0
  },
  {
    "name": "Sara",
    "lv": 13.0
  },
  {
    "name": "Ben",
    "lv": 12.0
  }
]
```  
  
出力結果が10.0,15.0などの小数点表記になっているのはMemberクラスのlvがdecimalのためです。  
  
#### 大量のデータを解析する方法  
大量のデータを少ないメモリで処理するにはJsonTextReaderを使用します。  
  
```csharp
        static void Test3()
        {
            using (FileStream fs = new FileStream(@"c:\share\jsontest\test003.json", FileMode.Open, FileAccess.Read))
            using (StreamReader sr = new StreamReader(fs))
            using (JsonTextReader reader = new JsonTextReader(sr))
            {
                int cnt = 0;
                while (reader.Read())
                {
                    if (reader.TokenType == JsonToken.StartObject)
                    {
                        // Load each object from the stream and do something with it
                        JObject obj = JObject.Load(reader);
                        //Console.WriteLine(obj["num1"]);
                        ++cnt;
                    }
                }
                Console.WriteLine(cnt);
            }
        }
```  
  
ファイルストリームをJsonTextReaderに渡し、StartObjectを検知する度に処理を行うことで、すべてのデータをメモリに展開せずに解析が可能になります。  
  
#### RFCより緩い挙動  
RFC7159で認められていないJSONをJSONとしてみなす挙動が何点かあります。  
https://github.com/JamesNK/Newtonsoft.Json/issues/646  
  
たとえば以下のコードはjavascriptのJSON.parseでは例外となりますが、Json.NETでは通ってしまいます。  
  
```csharp
           // 二重引用符ではなく一重引用符を含むプロパティ
            dynamic json =  JsonConvert.DeserializeObject("{'a':1}");
            Console.WriteLine(json);
            // 引用符なしのプロパティ
            json = JsonConvert.DeserializeObject("{a:1}");
            Console.WriteLine(json);
            // NaN,Infinity, -Infinity
            json = JsonConvert.DeserializeObject("{\"a\":NaN}");
            Console.WriteLine(json.a);
            // 末尾のコンマ
            json = JsonConvert.DeserializeObject("{\"a\": 123,}");
            Console.WriteLine(json);
            // 空のコンマ
            json = JsonConvert.DeserializeObject("[1,,2]");
            Console.WriteLine(json);
            // 8進数
            json = JsonConvert.DeserializeObject("{\"a\": 010}");
            Console.WriteLine(json);
            // 16進数
            json = JsonConvert.DeserializeObject("{\"a\": 0x010}");
            Console.WriteLine(json);
```  
  
```text
{
  "a": 1
}
{
  "a": 1
}
NaN
{
  "a": 123
}
[
  1,
  null,
  2
]
{
  "a": 8
}
{
  "a": 16
}

```  
  
**2018年時点の仕様より多くのものを許可してしまっているということに注意しなければいけません。特に複数のプログラミング言語を使用している場合、注意が必要です。**  
  
例：C#で通ったJSONがnode.jsとかでは通らないとかがあり得る。  
  
## PowerShell  
環境：  
Windows10  
PSVersion  5.1.17134.765  
PSEdition  Desktop  
  
### ConvertFrom-Json と　ConvertTo-Json  
ConvertFrom-JsonはJSONをPowerShellのオブジェクトに変換し、ConvertTo-JsonはPowerShellのオブジェクトをJSONに変換します。  
  
**ConvertFrom-Json**  
https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/convertfrom-json?view=powershell-5.1  
  
**ConvertTo-Json**  
https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/convertto-json?view=powershell-5.1  
  
[JavaScriptSerializer class](https://docs.microsoft.com/en-us/dotnet/api/system.web.script.serialization.javascriptserializer?view=netframework-4.8)を用いて実装しているとのことですが、これはC#で出てきたJson.NET を使用しているようです。  
  
```powershell
$json = Get-Content  -Encoding UTF8 c:\share\jsontest\test001.json  | ConvertFrom-Json
Write-Host $json.GetType()
Write-Host $json
Write-Host $json.num1.GetType()
Write-Host $json.num1
Write-Host $json.num2.GetType()
Write-Host $json.num2
Write-Host $json.num3.GetType()
Write-Host $json.num3
Write-Host $json.num4.GetType()
Write-Host $json.num4
Write-Host $json.num5.GetType()
Write-Host $json.num5
Write-Host $json.obj1.GetType()
Write-Host $json.obj1
Write-Host $json.obj1.a
Write-Host $json.obj1.b.GetType()
Write-Host $json.obj1.b
Write-Host $json.obj1.b.c.GetType()
Write-Host $json.obj1.b.c
Write-Host $json.obj1.b.c.d1.GetType()
Write-Host $json.obj1.b.c.d1
Write-Host $json.ary1.GetType()
Write-Host $json.ary1
Write-Host $json.ary1[0]

# https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/convertto-json?view=powershell-5.1
$str = ConvertTo-Json -depth 100 -InputObject $json
Write-Host $str
```  
  
  
実行結果は以下のようになります。  
  
```text
System.Management.Automation.PSCustomObject
@{null=; num1=99999; num2=123.45; num3=-123.45; num4=1000; num5=0.001; bool1=True; bool2=False; str1=abあいうえおcdef; str2=str2:
\       /ab0ぁ; obj1=; ary1=System.Object[]; ary2=System.Object[]}
System.Int32
99999
System.Decimal
123.45
System.Decimal
-123.45
System.Double
1000
System.Double
0.001
System.Management.Automation.PSCustomObject
@{a=125; b=}
125
System.Management.Automation.PSCustomObject
@{c=}
System.Management.Automation.PSCustomObject
@{d1=; d2=12345}
System.Management.Automation.PSCustomObject
@{v=1234}
System.Object[]
1 2 test @{a=5.3}
1
{
    "null":  null,
    "num1":  99999,
    "num2":  123.45,
    "num3":  -123.45,
    "num4":  1000,
    "num5":  0.001,
    "bool1":  true,
    "bool2":  false,
    "str1":  "abあいうえおcdef",
    "str2":  "str2:\n\r\\\t/abc\b0ぁ",
    "obj1":  {
                 "a":  125,
                 "b":  {
                           "c":  {
                                     "d1":  {
                                                "v":  1234
                                            },
                                     "d2":  12345
                                 }
                       }
             },
    "ary1":  [
                 1,
                 2,
                 "test",
                 {
                     "a":  5.3
                 }
             ],
    "ary2":  [
                 1,
                 2,
                 3
             ]
}
```  
  
重複しているnum1は後方が優先されており、JSONのObject型はSystem.Management.Automation.PSCustomObject、JSONのArray型はSystem.Object[]に変換されています。  
JSONのNumber型はInt32,Double,Decimalのいずれかの型に変換されていることが確認でき、型の取り扱いに注意が必要と考えられます。  
  
## Excel VBA  
環境：  
Windows10  
Office16(32bit)  
  
### JScriptを用いた変換  
UTF8のJSONファイルをADODB.Streamで読み込み、それをJScriptのオブジェクトに変換する方法が、下記のページに紹介されていました。  
  
**How to parse JSON with VBA without external libraries?**  
https://stackoverflow.com/questions/19360440/how-to-parse-json-with-vba-without-external-libraries  
  
この方法は正常に動作しないケースがあります。  
  
```vb
Private Function ReadUtf8File(ByVal path As String) As String
    Dim objStream, strData
    
    Set objStream = CreateObject("ADODB.Stream")
    
    objStream.Charset = "utf-8"
    objStream.Open
    Call objStream.LoadFromFile(path)
    
    ReadUtf8File = objStream.ReadText()
    
    objStream.Close
    Set objStream = Nothing
End Function

Sub Test1_1()
    ' https://stackoverflow.com/questions/19360440/how-to-parse-json-with-vba-without-external-libraries
    Dim json As Object
    Dim scriptControl As Object

    Set scriptControl = CreateObject("MSScriptControl.ScriptControl")
    scriptControl.Language = "JScript"

    Dim contents As String
    contents = ReadUtf8File("C:\\share\\jsontest\\test001.json")

    Set json = scriptControl.Eval("(" + contents + ")")
    Debug.Print json
    
    ' 配列が操作できない

End Sub
```  
  
**配列の操作ができない**  
![image.png](/image/c4a51a78-83a0-2b5f-2436-e4fc349574ff.png)  
  
ウォッチ式を見る限り配列として取得されているようだが、配列の操作が一切できない。  
  
**array, objectを含まないJSONが解析できない**  
  
```vb
Sub Test1_2()
    Dim json As Object
    Dim scriptControl As Object

    Set scriptControl = CreateObject("MSScriptControl.ScriptControl")
    scriptControl.Language = "JScript"

    Dim contents As String
    contents = "12345"

    ' エラー
    Set json = scriptControl.Eval("(" + contents + ")")
End Sub
```  
  
### VBA-JSON  
VBA-JSONはVBAでJSONの変換を可能としたライブラリです。  
https://github.com/VBA-tools/VBA-JSON  
  
使用方法としては以下の通りです。  
１． 「Microsoft Scripting Runtime」を参照設定すること  
２．　[JsonConverter.bas](https://github.com/VBA-tools/VBA-JSON/blob/master/JsonConverter.bas)を取り込む  
３．　下記のような実装を行う  
  
```vb
Sub test2_1()
    Dim contents As String
    contents = ReadUtf8File("C:\\share\\jsontest\\test001.json")
    Dim Parsed As Dictionary
    Set Parsed = JsonConverter.ParseJson(contents)
    Debug.Print Parsed.Item("num1")
    Debug.Print Parsed.Item("num2")
    Debug.Print Parsed.Item("null")
    Debug.Print Parsed.Item("obj1").Item("a")
    Debug.Print Parsed.Item("ary1").Count
    Debug.Print Parsed.Item("ary1").Item(1)
    Debug.Print Parsed.Item("ary1").Item(2)
    Debug.Print Parsed.Item("ary1").Item(3)
End Sub
```  
  
先ほどと違い配列データも正常に取得が可能になっています。  
  
```text
Test2_1
 99999 
 123.45 
Null
 125 
 4 
 1 
 2 
test
```  
  
ただし、最新の仕様に対応していないため、以下のようにobjectまたはarrayを含まないJSONについて仕様通り動作しません。  
  
```vb
Sub test2_2()
    Dim contents As String
    contents = "12345"
    Dim Parsed As Dictionary
    ' エラー
    Set Parsed = JsonConverter.ParseJson(contents)
End Sub

```  
  
# さいごに  
今回は謙虚な気持ちでJSONについて調べなおしてみました。  
  
わかってたけど なにもわかっちゃいなかったことも まれによくあるので、わかったつもりになっているモノこそ基礎から見直してみることも必要かもしれません。  
  
さもないと慢心の精霊に取りつかれて「JSONはobjectまたはarrayを必ず含む必要ある！どや！( ･`ー･´)」とか言って老害っぷりをさらすことになります。  
  
# その他・参考  
**Wikipedia**  
https://ja.wikipedia.org/wiki/JavaScript_Object_Notation  
https://en.wikipedia.org/wiki/JSON  
  
**how to parse a large, Newline-delimited JSON file by JSONStream module in node.js?**  
https://stackoverflow.com/questions/15121584/how-to-parse-a-large-newline-delimited-json-file-by-jsonstream-module-in-node-j  
  
**Json.NETドキュメント**  
https://www.newtonsoft.com/json/help/html/Introduction.htm  
