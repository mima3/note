# はじめに  
ひさしぶりにPythonで久しぶりにHTMLを出力したくなったのでテンプレートについて調べます。  
  
**環境：**  
Python 3.7.4  
  
# 標準ライブラリでの文字列の書式の扱い  
まず標準で使える範囲で文字列の書式をどう扱えるか調べます。  
これについては下記の素晴らしいまとめが存在します。  
  
**Python String Formatting Best Practices**  
https://realpython.com/python-string-formatting/  
  
ここで紹介されている方法は以下の通りです。  
  
 - 古い形式の文字列の書式  
 - 新しい形式の文字列の書式  
 - PEP498で定義された書式設定方法  
 - テンプレート文字列  
  
  
## 古い形式の文字列の書式  
昔ながらの文字列の書式の指定方法です。  
  
```python
name = 'アンジュ'
age = 18

print('古いやり方--------------------------------------')
outstr = '%s は %d歳' %(name, age)
print (outstr)

outstr = '%s は 0x%x歳' %(name, age)
print (outstr)

outstr = '%(name)s は %(age)d歳' % {'name':name, 'age': age}
print (outstr)

outstr = '%(name)s は 0x%(age)x歳' % {'name':name, 'age': age}
print (outstr)
```  
  
[printf 形式の文字列書式化](https://docs.python.org/ja/3/library/stdtypes.html#old-string-formatting)に以下のような記述があるので今となっては別の方法を選択した方がいいでしょう。  
  
>**注釈** ここで解説されているフォーマット操作には、(タプルや辞書を正しく表示するのに失敗するなどの) よくある多くの問題を引き起こす、様々な欠陥が出現します。 新しい フォーマット済み文字列リテラル や str.format() インターフェースや テンプレート文字列 が、これらの問題を回避する助けになるでしょう。 これらの代替手段には、それ自身に、トレードオフや、簡潔さ、柔軟さ、拡張性といった利点があります。  
  
## 新しい形式の文字列の書式  
[str.format](https://docs.python.org/ja/3/library/string.html#string-formatting)による書式の設定は、Python3から導入され後にPython2.7に移植されました。  
  
  
```python
name = 'アンジュ'
age = 18

print('str.format--------------------------------------')
outstr = '{} は {}歳'.format(name,age)
print (outstr)
outstr = '{} は 0x{:x}歳'.format(name,age)
print (outstr)
outstr = '{name} は {age}歳'.format(name=name, age=age)
print (outstr)
outstr = '{name} は 0x{age:x}歳'.format(name=name, age=age)
print (outstr)
```  
  
## PEP498で定義された書式設定方法  
[PEP498](https://www.python.org/dev/peps/pep-0498/)で定義された書式の設定方法で[Python3.6から追加されました](https://dbader.org/blog/cool-new-features-in-python-3-6)。  
これはES2015/ES6で追加された[JavaScriptのテンプレートリテラルに似ています](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)。  
  
```python
name = 'アンジュ'
age = 18

print('f-strings--------------------------------------')
outstr = f'{name} は {age}歳'
print (outstr)
outstr = f'{name} は 0x{age:x}歳'
print (outstr)
outstr = f'{name} は {10+8}歳'
print (outstr)
```  
  
## テンプレート文字列  
[テンプレート文字列](https://docs.python.org/ja/3.7/library/string.html#template-strings)は[PEP292](https://www.python.org/dev/peps/pep-0292/)で解説されている単純な文字列置換を行います。  
  
```python
from string import Template
tpl = Template('$name は $age歳')

name = 'アンジュ'
age = 18

print('template--------------------------------------')
print(tpl.substitute(name=name, age=age))
# %xなどの書式は使えない
print(tpl.substitute(name=name, age=hex(age)))
```  
  
ユーザからの入力を受け取る場合はテンプレート文字列を使用した方が望ましいです。  
たとえばformat()を使用して以下の実装をしたとしましょう。  
  
```python
SECRET = '秘密の文字列'

class Error:
  def __init__(self):
    pass

err = Error()

# ユーザから以下のような入力を受け付けたとする
user_input = '{error.__init__.__globals__[SECRET]}'
# 出したくない文字列が表示される
print(user_input.format(error=err))
```  
  
ユーザ入力に対してformatを実行すると出したくない文字列にアクセスできたりします。  
  
同様の実装をテンプレート文字列を使用すると以下のようになります。  
  
```python
SECRET = '秘密の文字列'

class Error:
  def __init__(self):
    pass

err = Error()

# ユーザが以下を入力したとする
user_input = '${error.__init__.__globals__[SECRET]}'

# エラーになります
# ValueError: Invalid placeholder in string: line 1, col 1
print(Template(user_input).substitute(error=err))
```  
  
この問題は[Be Careful with Python's New-Style String Format](http://lucumr.pocoo.org/2016/12/29/careful-with-str-format/)に記載されています。  
  
# HTML用のテンプレートライブラリ  
単純な文字の置換であれば今まで標準のライブラリで十分ですが、繰り返しとかを定義したい場合はHTML用のテンプレートライブラリを探す必要があります。  
  
HTMLを自動生成するためのテンプレートライブラリとして以下のページで比較がなされています。  
  
**3 Python template libraries compared**  
https://opensource.com/resources/python/template-libraries  
  
このページで紹介されている方法は以下の３つになります。  
  
 - [Mako](https://www.makotemplates.org/)  
 - [Jinja2](https://jinja.palletsprojects.com/en/2.10.x/)  
 - [Genshi](https://genshi.edgewall.org)  
  
  
## StackOverflowでの質問件数の比較  
Mako,Jinja2,Genshiともに、機能や環境的には今回の使用目的にはあっているので、よく使われているライブラリを使用することにします。  
そのため、StackOverflowでの質問件数を調べます。  
  
2019/10/18時点  
![image.png](/image/5b31076b-33e5-03bb-fada-c2922121b0a3.png)  
  
![image.png](/image/cf354f9b-5191-0e94-2f31-1b88f96abf21.png)  
  
![image.png](/image/e3a88e7e-bcca-50db-830b-a077f30b1766.png)  
  
  
**結果：**  
  
|名前|検索結果|  
|:--|:--|  
|Mako|1937|  
|Jinja2|16980|  
|Genshi|363|  
  
Jinja2の圧勝のようです。  
  
なお、以下のページでタグごとのトレンドを比較できますが、Jinja2以外タグがなかったので今回は採用しませんでした。  
https://insights.stackoverflow.com/trends  
  
## Jinja2を使用してみる  
### インストール方法  
インストール可能な条件は以下の通りです。  
  
>Jinja2 works with Python 2.6.x, 2.7.x and >= 3.3. If you are using Python 3.2 you can use an older release of Jinja2 (2.6) as support for Python 3.2 was dropped in Jinja2 version 2.7.  
  
[Introduction - Jinja Documentation](https://jinja.palletsprojects.com/en/2.10.x/intro/)  
  
  
2019年10月18日時点では以下のコマンドでJinja2-2.10.3がインストールされるようです。  
  
```
pip install Jinja2
# アップデートの場合
# pip install -U Jinja2
```  
  
### 使ってみる  
テンプレートの書き方は以下参考。  
https://jinja.palletsprojects.com/en/2.10.x/templates/  
  
  
#### 単純な例  
  
```python

from jinja2 import Template
template = Template('Hello {{ name }}!')
print (template.render(name='アンジュ'))
```  
  
**出力結果**  
  
```
Hello アンジュ!
```  
  
#### ループしてみる  
  
```python
from jinja2 import Template
html = '''
<!DOCTYPE html>
<html lang="en">
<head>
    <title>My Webpage</title>
</head>
<body>
    <ul id="navigation">
    {% for item in navigation %}
        <li><a href="{{ item.href }}">{{ item.caption}} </a></li>
    {% endfor %}
    </ul>

    <h1>My Webpage</h1>
    {{ a_variable }}
    
</body>
</html>
'''
template = Template(html)
data = {
  'a_variable' : 'わっふる',
  'navigation' : [
    {'href':'http://hogehoge1', 'caption': 'test1'},
    {'href':'http://hogehoge2', 'caption': 'test2'}
  ]
}
print (template.render(data))
```  
  
**出力結果**  
  
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>My Webpage</title>
</head>
<body>
    <ul id="navigation">

        <li><a href="http://hogehoge1">test1 </a></li>

        <li><a href="http://hogehoge2">test2 </a></li>

    </ul>

    <h1>My Webpage</h1>
    わっふる

</body>
</html>
```  
  
#### テンプレートを外部ファイルに出してみる  
  
```jinja/001.tpl
<!DOCTYPE html>
<html lang="en">
<head>
    <title>My Webpage</title>
</head>
<body>
    <ul id="navigation">
    {% for item in navigation %}
        <li><a href="{{ item.href }}">{{ item.caption}} </a></li>
    {% endfor %}
    </ul>

    <h1>My Webpage</h1>
    {{ a_variable }}
    
</body>
</html>
```  
  
```python
from jinja2 import Environment, FileSystemLoader
data = {
  'a_variable' : 'わっふる',
  'navigation' : [
    {'href':'http://hogehoge1', 'caption': 'test1'},
    {'href':'http://hogehoge2', 'caption': 'test<>2'}
  ]
}
from jinja2 import Environment, FileSystemLoader
env = Environment(loader = FileSystemLoader('./templates'))
template = env.get_template('001.tpl')
print (template.render(data))
```  
  
**出力結果**  
  
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>My Webpage</title>
</head>
<body>
    <ul id="navigation">

        <li><a href="http://hogehoge1">test1 </a></li>

        <li><a href="http://hogehoge2">test<>2 </a></li>

    </ul>

    <h1>My Webpage</h1>
    わっふる

</body>
</html>
```  
  
#### HTMLエスケープの方法  
与えるデータに< , > , &, "が存在する場合のエスケープ方法には２種類ある。  
  
##### 手動エスケープ  
手動エスケープは[escapeフィルタ](https://jinja.palletsprojects.com/en/2.10.x/templates/#escape)をテンプレートの変数に指定する方法です。  
  
```jinja:templates/002.tpl
<!DOCTYPE html>
<html lang="en">
<head>
    <title>My Webpage</title>
</head>
<body>
    <ul id="navigation">
    {% for item in navigation %}
        {# | e または | escape でエスケープ可能 #}
        <li><a href="{{ item.href }}">{{ item.caption | e}} </a></li>
    {% endfor %}
    </ul>

    <h1>My Webpage</h1>
    {{ a_variable }}
    
</body>
</html>
```  
  
```python
from jinja2 import Environment, FileSystemLoader
data = {
  'a_variable' : 'わっふる',
  'navigation' : [
    {'href':'http://hogehoge1', 'caption': 'test&!1'},
    {'href':'http://hogehoge2', 'caption': 'test<>2'}
  ]
}
from jinja2 import Environment, FileSystemLoader
env = Environment(loader = FileSystemLoader('./templates'))
template = env.get_template('001.tpl')
print (template.render(data))
template = env.get_template('002.tpl')
print (template.render(data))
```  
  
**結果**  
エスケープしていないテンプレートとエスケープしたテンプレートを使用した場合の出力結果は以下になります。  
  
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>My Webpage</title>
</head>
<body>
    <ul id="navigation">

        <li><a href="http://hogehoge1">test&!1 </a></li>

        <li><a href="http://hogehoge2">test<>2 </a></li>

    </ul>

    <h1>My Webpage</h1>
    わっふる

</body>
</html>

<!DOCTYPE html>
<html lang="en">
<head>
    <title>My Webpage</title>
</head>
<body>
    <ul id="navigation">


        <li><a href="http://hogehoge1">test&amp;!1 </a></li>


        <li><a href="http://hogehoge2">test&lt;&gt;2 </a></li>

    </ul>

    <h1>My Webpage</h1>
    わっふる

</body>
</html>
```  
  
##### 自動エスケープ  
Environmentを実行時にautoescapeを指定することでテンプレートにエスケープ処理を記載する必要がなくなります。  
  
```python
from jinja2 import Environment, FileSystemLoader
data = {
  'a_variable' : 'わっふる',
  'navigation' : [
    {'href':'http://hogehoge1', 'caption': 'test&!1'},
    {'href':'http://hogehoge2', 'caption': 'test<>2'}
  ]
}
env = Environment(loader = FileSystemLoader('./templates'), autoescape = True)
template = env.get_template('001.tpl')
print (template.render(data))
```  
  
# まとめ  
単純な文字の書式を使う場合は以下の通りがいいようです。  
  
 - 書式文字列に対してユーザ入力ある場合  
    - テンプレート文字列  
 - ユーザ入力ない場合  
    - Python3.6以上  
        - PEP498で定義された書式設定方法  
    - Python3.6未満  
        - 新しい形式の文字列の書式  
  
HTMLのテンプレートライブラリとしてはJinja2がよく使われているようです。  
  
