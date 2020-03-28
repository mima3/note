# 概要  
RadonはPythonのメトリックスを計測するプログラムである。  
https://github.com/rubik/radon  
  
これにより、Pythonのコードの複雑度や行数を取得することができる。  
複雑度を計測することで、ソースコードの潜在的リスクを明確にして、リファクタリングやテストをすべき対象を明示できる。  
  
  
# インストール方法  
  
python2.xまたはPython3.xで下記を実行する。  
  
```
easy_install easy_install radon
```  
  
# 使い方  
## Cyclomatic Complexityの計測  
ccコマンドを使用することでCyclomatic Complexityの計測が行える。Cyclomatic Complexityは制御文が多いほど高くなる。  
詳細は下記を参考のこと。  
https://radon.readthedocs.org/en/latest/intro.html#cyclomatic-complexity  
  
  
 __使用例：__   
  
```
radon cc -s ".\*.py"
```  
  
![無題.png](/image/3ab4e165-ce47-4d48-dc15-846530314f76.png)  
  
 __出力内容：__   
最初の記号はブロックタイプを表す  
  
|記号|説明|  
|:---|:---|  
|F|関数|  
|M|メソッド|  
|C|クラス|  
  
  
次の数値は計測対象のブロックの開始行列を表す  
  
```
行:列
```  
  
次は関数名などのブロック名を表す。  
  
「-」以降はランクとCyclomatic Complexityの値となる  
  
ランクはCyclomatic Complexityの値によって決まる。  
  
|CCの値|ランク|リスク|  
|:-----|:-----|:-----|  
|1 - 5 |A|low - 単純なブロック|  
|6 - 10|B|low - よく構造化され、安定したブロック|  
|11 - 20|C|moderate - 少し複雑なブロック|  
|21 - 30|D|more than moderate - より複雑なブロック|  
|31 - 40|E|high - 複雑なブロック、憂慮すべき|  
|41+ |F|very high - エラーが発生しやすい、不安定なブロック|  
  
 __オプション:__   
ccコマンドで使用できるオプションは次の通り  
  
|Option|説明|  
|:-----|:---|  
|-s, --show|Cyclomatic Complexityを表示する。|  
|-n, --min|表示する最小のランクを設定する。この引数の後に A～Fを指定|  
|-x, --max|表示する最大のランクを設定する。この引数の後に A～Fを指定|  
|-a, --average|Cyclomatic Complexityの平均を最後に表示する。この平均は-nや-xによるフィルタに影響をうける|  
|--total-average|すべてのデータの平均をうける-aと異なり、フィルタの影響を受けない|  
|-e, --exclude|解析の対象から外すファイルのパスをコンマ区切りで指定する|  
| -i, --ignore|無視するディレクトリをコンマ区切りで指定する。無視したディレクトリ以下は検査しない|  
|-o, --order|並び順を指定する.<BR>SCORE 複雑度順<BR>LINES 行数順<BR>ALPHA ブロック名順|  
| -j, --json|結果をJSONで出力する|  
|--no-assert|複雑度を計算する場合にasser()命令はカウントしない|  
  
## Maintainability Indexの計測  
miコマンドを使用することでMaintainability Indexを計測する。MaintainabilityはCyclomatic Complexityと行数から計算され、100を最高に高いほど、保守しやすい。  
  
詳細は以下を参照  
https://radon.readthedocs.org/en/latest/intro.html#maintainability-index  
  
 __使用例：__   
  
```
radon mi -s ".\*.py"
```  
  
 __出力内容：__   
  
```
C:\dev\py33>radon mi -s "test.py"
test.py - A (58.76)
```  
  
ファイル名、ランク、MI scoreを表示する  
  
ランクについては下記の通り  
  
|MI score|Rank|Maintainability|  
|:-------|:---|:--------------|  
|100 - 20|A|非常に高い|  
|19 - 10|B|中程度|  
|9 - 0|C|非常に低い|  
  
 __オプション:__   
miコマンドで使用できるオプションは次の通り  
  
|Option|説明|  
|:-----|:---|  
|-s, --show|Maintainability Indexを表示する。|  
|-e, --exclude|解析の対象から外すファイルのパスをコンマ区切りで指定する|  
|-i, --ignore|無視するディレクトリをコンマ区切りで指定する。無視したディレクトリ以下は検査しない|  
| -m, --multi|複数行の文字列をコメントとみなして数えない|  
  
# rawデータの計測  
rawコマンドは以下の数値を出力する。  
  
・ __LOC__ ：総行数  
・ __LLOC__ :Logical LOC。空行とかコメント行を除いたもの  
・ __SLOC__ :Source LOC .空行を除いたもの。  
・ __comments__ :コメント行  
・ __multi__ :複数行の文字列。コメントとみなされることがある  
・ __blank__ : 空行  
  
 __オプション:__   
rawコマンドで使用できるオプションは次の通り  
  
|Option|説明|  
|:-----|:---|  
|-s, --summary|計測した集計結果を表示する|  
|-e, --exclude|解析の対象から外すファイルのパスをコンマ区切りで指定する|  
|-i, --ignore|無視するディレクトリをコンマ区切りで指定する。無視したディレクトリ以下は検査しない|  
| -j, --json|結果をJSONで出力する|  
  
# プログラムからの利用  
radonモジュールをpythonのコードにimportすることでpython上からradonが利用できる。  
  
例：  
  
```py
import radon
from radon import raw
ret = radon.raw.analyze("""
if a==1:
  print (a)
if a==2:
  print (a)
""")
print(ret)
```  
  
詳細についてはhelpで調べるか下記参照  
https://radon.readthedocs.org/en/latest/api.html  
