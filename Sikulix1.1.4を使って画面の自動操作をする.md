# 概要  
Sikuli（シークリー）は画像認識を用いて、Windows/Mac/Linuxの画面を操作することが可能です。  
http://sikulix.com/  
なお、語源としてはインディアンの文化の奉納物で未知のものを見て理解する力という意味とのことです。  
  
今回は1.1.4を使って検証しますが、最新バージョンは現時点で色々とドキュメントと実装の食い違いがあったり、最新の[コード](https://github.com/RaiMan/SikuliX1)をビルドしないと動かなかったりするので、画像認識で自動操作したいだけの人は1.1.3を使用した方がいいと思います。  
また、今回は「[ｱｶﾈﾁｬﾝｶﾜｲｲﾔｯﾀ-](https://qiita.com/mima_ita/items/d4655de865f30bb51c65)」よりちょっと詳しく記載してます。  
  
  
## 現在のリリース状況  
2019/10時点のリリースの状況は以下の通りになっています。  
![image.png](/image/79ab68fa-5ad6-16ae-2536-1e7a19266873.png)  
  
https://launchpad.net/sikuli/+series  
  
本記事は1.1.4時点の記事なので注意してください。  
※以下を読んで軽く使った感じだと、1.1.4と2.0で大きな違いは感じませんでしたが・・・  
https://sikulix-2014.readthedocs.io/en/latest/news.html  
  
  
# 環境構築  
## 2019/5時点の最新リリースの導入方法  
現時点で最新の1.1.4のWindows10への導入方法について記載します。  
  
(1)Java8以上のJDK(**64bit**) を入手します  
以下のいずれかから入手します。今回の実験では「Pleiades All in One」に入っていたJDK8を使用しました。  
  
**Oracle/Sun**  
https://www.oracle.com/technetwork/java/javase/downloads/index.html  
  
**OpenJDK**  
https://openjdk.java.net/  
  
**Pleiades All in One**  
http://mergedoc.osdn.jp/  
  
(2)下記のページより必要なJARを取得し同一フォルダに配置します  
https://raiman.github.io/SikuliX1/downloads.html  
  
文中の以下のリンクからJARを取得します。  
・Download the ready to use sikulixapi.jar  
・The Jython interpreter 2.7.1 for python scripting (the default)  
・The JRuby interpreter 9.x for ruby scripting  
・Download the ready to use sikulix.jar (SikuliX IDE)  
  
※sikulix.jarを起動後、jruby,jythonのjarは以下のフォルダに移動します。  
・C:\Users\ユーザ\AppData\Roaming\Sikulix\Extensions  
  
(3)IDEの起動  
  
**1.1.4時点**  
  
```
SET JAVA_HOME=C:\pleiades201904\java\8
"%JAVA_HOME%\bin\java.exe"  -jar sikulix.jar
```  
  
**2.0時点**  
  
```
SET JAVA_HOME=C:\pleiades201904\java\8
"%JAVA_HOME%\bin\java.exe"  -jar sikulix-2.0.0.jar
```  
  
  
## 最新ソースの使用方法  
（１）下記のGitHubから最新をダウンロードします。  
https://github.com/RaiMan/SikuliX1  
  
（２）「Pleiades All in One」にてMavenのプロジェクトとしてインポートします。  
  
（３）JDK12でビルド（実行→Maven Install）をおこなうことでtargetフォルダにjarが作成されるので、それを利用する。  
  
# 簡単なチュートリアル  
## Hello World  
伝統的なHello WorldをSikulixでやってみます。  
メモ帳を立ち上げて、「はろーわーるど！」と打ち込みます。  
  
手で起動する場合を考えてみましょう。  
たとえば、以下のような手順になると思います。  
  
（１）メモ帳を起動する  
（２）メモ帳が起動するまでまつ  
（３）メモ帳のテキストエリアをクリックする  
（４）「はろーわるど！」と入力する。  
  
（１）メモ帳を起動する。  
sikulixではアプリケーションを起動する場合は、以下のようなスクリプトを記述します。  
  
```
app = App('アプリケーションのパス')
app.open()
```  
  
メモ帳の場合は以下のようになります。  
  
![image.png](/image/25d6036e-04f7-5740-f0e1-b854b3e695d2.png)  
  
（２）メモ帳が起動するまでまつ  
以下のようなメモ帳が起動するまで待機するにはどうしたらいいでしょうか？  
![image.png](/image/f25a9b01-1b75-5e7a-9547-19527df46e3b.png)  
  
まず、「スクリーンショットを撮る」をクリックします。  
![image.png](/image/58b32e85-91e9-e1e3-696b-579c0ad28fe6.png)  
  
ディスクトップ上で矩形を選択できるようになりますのでメモ帳タイトルバーが入るように矩形を選択すると、IDEに以下のような画像が表示されます。  
  
![image.png](/image/a7bf0713-d466-b159-f478-c0442219be15.png)  
  
選択した画像が表示されるまで待つために「wait()」コマンドを以下のように使用します。  
  
![image.png](/image/5f217621-3606-d697-f9ed-bb74135f0968.png)  
  
なお、昔のバージョンではコマンドリストなるものが存在したそうですが、1.1.4では消えたので、おとなしく手でコマンドを入力してください。  
  
（３）メモ帳のテキストエリアをクリックする  
画像の位置をクリックするには「click()」コマンドを使用します。  
waitと同じように以下のように記載します。  
  
![image.png](/image/014266e8-316d-ca00-33e5-15476c33f776.png)  
  
これで画像をクリックするようになりましたが、クリックする位置はデフォルトで中央になっています。これを変更するには、click()中の画像をクリックしてください。  
  
パターン設定というポップアップが表示されると思います。  
![image.png](/image/7c8d0683-9257-09ba-c844-f56357fb1d94.png)  
  
ここで、「ターゲットオフセット」タブを選択してください。  
![image.png](/image/2958c891-3b30-4c33-6f5f-6740152d9d71.png)  
  
この画面でクリック位置を設定できます。  
マウスホイールで拡大、縮小ができ、画像上を押下して、自動操作中にクリックしたい位置を指定できます。  
「ターゲットオフセット」画面では、クリック位置を十字カーソルで表現しています。現在は「書式」メニューあたりをクリックするようになっているので、以下のようにテキストエリアをクリックしてみましょう。  
  
![image.png](/image/0096e3be-d12e-961c-263d-5136a6308735.png)  
  
クリック位置を変更すると、画面下部のターゲットオフセットの値が０から変化したと思います。  
クリック位置を確定するために「OK」ボタンを押下してください。  
  
![image.png](/image/d8b211bc-2e17-07f3-83f2-bf23adb5ffc1.png)  
IDE上の画像にクリック位置が赤いカーソルで表現されるようになりました。  
  
（４）「はろーわるど！」と入力する。  
キーボード入力には「type」コマンドと「paste」コマンドの２種類が存在します。  
日本語入力を行いたい場合は「paste」コマンドを利用します。「paste(u'はろーわーるど！')」とIDEに入力します。  
![image.png](/image/e622a15c-79c5-4325-418b-9834aed5d98f.png)  
  
では、記載したスクリプトを起動して確認してみましょう。  
スクリプトを起動するには![image.png](/image/68286367-1dcc-766e-9888-2a0888cc8697.png)をクリックします。  
  
![image.png](/image/e67486ce-dcce-65b5-d869-ea266158b5bb.png)  
  
「Run Immediately」または「Save all and Run」が表示されます。保存しないですぐ実行するか、保存してから実行するかを選択できますが、今回は保存しないで実行するために「Run Immediately」を選択します。  
  
これでメモ帳が起動して「はろーわーるど！」と表示されると思います。  
  
その他、公式のチュートリアルは下記を参照してください。  
https://sikulix-2014.readthedocs.io/en/latest/index.html#tutorials  
  
# IDEの使い方  
  
## ファイル  
![image.png](/image/6678aa0f-d940-beac-51b3-87c2d9a6cad2.png)  
  
  
### 新規作成  
新規にファイルを作成します。  
![image.png](/image/c4234c9f-c56f-1d9b-5f20-822d47676b0e.png)  
  
この時点では、「C:\Users\ユーザ名\AppData\Local\Temp\」以下に一時フォルダが作成されており、保存時にその内容を移動することになります。  
  
### 開く  
![image.png](/image/50a5feb8-bf81-5efc-4c56-f6bad48c6961.png)  
  
以下のファイルを開きます。  
  
- sikuli拡張子のフォルダ  
- py拡張子のファイル  
- その他テキストファイル  
  
・sikuli拡張子のフォルダはスクリプトファイルとスクリプトで用いる画像ファイルが格納されているフォルダです。  
以下はさきほどのHelloWorldで作成した内容を保存したsikuli拡張子フォルダの中身になります。  
![image.png](/image/915c051f-8bab-6abd-aee6-b4e51e23b043.png)  
  
  
・py拡張子ファイルはpythonのファイルを選択します。このファイルも実行が可能です。  
  
・その他テキストファイルはただのテキストファイルとしてひらくため、実行することはできません。  
  
### 保存、名前を付けて保存、save all  
・「保存」は現在編集中のスクリプトまたはテキストを上書き保存します。ただし、新規作成直後については名前を付けて保存と同じ処理になります。  
  
・「名前を付けて保存」は現在編集中のスクリプトを保存場所を選択して保存します。  
  
・「save all」は現在開いている全てのタブの内容を上書き保存します。  
  
なお、保存時に使われていない画像ファイルは削除されます。  
この挙動は環境設定により変更可能です。  
  
  
### 実行ファイルとして保存  
skl拡張子としてエクスポートをします。  
エクスポートしたファイルは別のスクリプトから利用できます。  
  
たとえば以下のスクリプトを含むsklを作成します。  
  
**myModule.skl**  
```python:myModule.skl
from sikuli import *
def myFunction(s):
    Debug.log(1, 'myFunction%s'  ,s)

print ('test')
```  
  
このファイルで定義した関数myFunctionは別のスクリプトから以下のように利用できます。  
  
```python
addImportPath(makePath(getParentFolder() , 'myModule.skl'))
# myModule読み込み
import myModule
# 別モジュール
myModule.myFunction('xxx')
```  
  
これによりスクリプトファイルの部品化が行えるようになります。  
なお、sklファイルはsikuli拡張子のフォルダをzipで圧縮したものです。  
  
※以下のヘルプによるとコマンドラインよりsklファイルの実行もできたようですが、1.1.4の段階ではコマンドラインからは実行できないようです。  
https://sikulix-2014.readthedocs.io/en/latest/faq/010-command-line.html  
  
  
### Export as jar  
・Jarファイルとして出力します。  
  
・ドキュメント的には単独で動作可能に見えますが、実際は単独で動作しません。  
https://github.com/RaiMan/SikuliX1/issues/33  
  
これは1.1.4で取り消された機能のようです。  
https://sikulix-2014.readthedocs.io/en/latest/newsbugs.html  
  
・JrubyではJarのExport自体実装されていないようです。  
https://answers.dogfood.paddev.net/sikuli/+question/678016  
  
### Open Specials Files  
特殊な設定ファイルを修正します。  
なお、2019/4リリース時点で動作しません。  
  
GitHub版のコードをコンパイルして実行すると以下のような画面が表示されます。  
![image.png](/image/b8ea0f46-24dc-a40b-b7c9-67f37f7aa60a.png)  
  
|No |Name|Path|  
|:--|:----|:---|  
|1|Sikulix Global Options|C:\Users\(User)\AppData\Roaming\Sikulix\SikulixStore\SikulixOptions.txt|  
|2|SikuliX Extensions Options|C:\Users\(User)\AppData\Roaming\Sikulix\Extensions\extensions.txt|  
|3|SikuliX Additional Sites|C:\Users\(User)\AppData\Roaming\Sikulix\Lib\site-packages\sites.txt|  
  
#### Sikulix Global Options  
Settingsの初期値を設定できます。  
  
**SikulixOptions.txt**  
```text:SikulixOptions.txt
# key = value
Settings.LogTime = True
```  
  
上記を設定した場合の再起動直後のSettings.LogTimeがTrueになります。  
その他、Settingsの内容は下記を参照してください。  
  
https://sikulix-2014.readthedocs.io/en/latest/scripting.html#Settings  
  
  
#### SikuliX Extensions Options  
pythonやjythonのパスを指定できます。  
  
```text
# add absolute paths one per line, that point to other jars,
# that need to be available on Java's classpath at runtime
# They will be added automatically at startup in the given sequence

# empty lines and lines beginning with # or // are ignored
# delete the leading # to activate a prepared keyword line

# pointer to a Jython install outside SikuliX
# jython = c:/jython2.7.1/jython.jar

# the Python executable as used on a commandline
# activating will enable the support for real Python
# python = C:\Users\XXXXXX\AppData\Local\Programs\Python\Python36-32\python.exe

```  
  
pythonの指定の目的がいまいちわかりませんでした。  
※runScriptでpyファイルを実行しても同じ拡張子のjythonのスクリプトランナーが優先されて実行されてしまうので、動かないようにみえる。  
  
#### SikuliX Additional Sites  
起動時に自動的に「sys.path」の後ろに追加されるJytho,Python,SikuliXのパスを指定します。  
  
addImportPathを使わなくても以下のように設定することで使用することができるようになります。  
  
**sites.txt**  
```text:sites.txt
# add absolute paths one per line, that point to other directories/jars,
# where importable modules (Jython, plain Python, SikuliX scripts, ...) can be found.
# They will be added automatically at startup to the end of sys.path in the given sequence

# lines beginning with # and blank lines are ignored and can be used as comments
C:\tool\sikulix\myModule.sikuli
```  
  
```python
# myModule読み込み
import myModule
# 別モジュール
myModule.myFunction('xxx')
exit(1)
```  
  
### Reset IDE  
IDEを再起動します。  
環境設定を変更した場合に必要です。  
  
## 環境設定  
### 「画面キャプチャ」タブ  
![image.png](/image/f00b3e38-a36b-5073-dd6b-fa18aafcc1fb.png)  
  
  
画面キャプチャの設定を変更できます。  
「スクリーンショットを撮る」を実行した場合、デフォルトではタイムスタンプのファイルが作成されますが、ここを「オフ（手動入力）」とすることで「スクリーンショットを撮る」を実行する度に以下のような画面が表示されて任意のファイル名を設定できるようになります。  
  
![image.png](/image/d88a701d-01b7-0f28-8bee-23f344e965c2.png)  
  
### 「テキスト編集」タブ  
![image.png](/image/e3eb2e55-6a98-11ae-d244-2c6ebfd9da9c.png)  
  
テキストのタブ数やフォントを変更できます。  
  
### 「全般」タブ  
![image.png](/image/ac150a27-5232-c421-f197-0bf5c8b2dd4a.png)  
  
自動更新は未実装。  
Languageは言語を変更できます。  
  
### more options  
![image.png](/image/26ff88d8-fd2b-a62d-a285-9e385afd71ab.png)  
  
メッセージの表示レベルや、実行時に自動保存を行うか否かとかの設定ができます。  
  
## IDE上でスクリプトを実行する  
IDE上でスクリプトを実行するためには以下の３つの方法があります。  
  
(1)ツールバーのボタンから実行する  
![image.png](/image/767b1542-0c9f-078d-0fcf-0ef6e70885fd.png)  
  
(2)実行メニューから実行する  
![image.png](/image/90546044-36a9-9242-5327-be712573ac82.png)  
  
(3)スクリプトの行番号を右クリックして、コンテキストメニューから実行する  
![image.png](/image/296a228b-bad9-253e-658c-8f987c66c020.png)  
  
|メニュー|説明|  
|:------|:---|  
|実行   |スクリプトを先頭から実行する。スクリプトに変更がある場合は保存を要求される。|  
|スローモーションで実行|クリック時にSettings.SlowMotionDelayで指定した秒（デフォルト2秒)の時間をかけてゆっくり移動する。移動先にはマーカ![image.png](/image/e446b71e-2c6f-800d-c839-60b94cef2f55.png)が表示される。ただし1.1.4(2019/05)時点では[バグ](https://bugs.launchpad.net/sikuli/+bug/1816298)のため動作しない模様。GitHubの最新では期待通り動作する。|  
|run selection|選択しているコードのみを実行する。スクリプトに変更があっても保存を要求されない。|  
|run line|カーソルのある行のみ実行する。スクリプトに変更があっても保存を要求されない。|  
|run from line|カーソルのある行から実行する。スクリプトに変更があっても保存を要求されない。|  
|run to line|カーソルのあるところまで実行する。スクリプトに変更があっても保存を要求されない。|  
  
## ドキュメントタブのコンテキストメニュー  
![image.png](/image/7e18262e-88b5-f3f6-9070-d3205ba4d26d.png)  
  
### About  
![image.png](/image/ac4181e9-7a05-0670-f483-26453df8c6bf.png)  
  
ドキュメントの保存場所が表示されます。  
  
### Set Type  
使用する言語を選択できます。  
選択後は、ドキュメントの内容はクリアされるので、新規作成の直後に選ぶとよいでしょう。  
![image.png](/image/0176e7d0-0532-c359-3402-178bcb2154d0.png)  
  
以下の内容が選択できます。  
・jython  
・ruby  
・javascript  
・text（ただのプレーンテキストで実行はできません）  
  
### Insert Path  
以下のダイアログを表示してファイルのパスを選択できます。  
![image.png](/image/36b3586a-35ec-0dd7-3858-7037ed58a4a5.png)  
  
選択後はIDEに以下のようにパスが表示されます。  
![image.png](/image/0d2a5f6e-3d77-b984-bb05-ddf677f34e92.png)  
  
ただし、Windowsの場合、「\」はエスケープ文字なので「\\\\」としてあげないと、正常なパスになりません。  
  
  
### Move Tab  
タブの位置を移動します。  
  
(1)移動するタブを選択して「Move Tab」を選択  
![image.png](/image/b9b65458-90f0-053e-0777-7fca6521ddb3.png)  
  
(2)タブが消えます。  
![image.png](/image/e4e137ab-8cc6-a0fc-f521-0afe78db9374.png)  
  
(3)移動させたい場所のタブを選択して、コンテキストメニューを開き「Insert Right」または「Insert Left」を入力する。  
![image.png](/image/5962ca95-9604-6f92-c6a3-bbdf2a1a8a13.png)  
  
Insert Leftを選択した例：選択したタブの左側に移動する  
![image.png](/image/e655b9d5-5278-8e62-5773-fe7d48141f9d.png)  
  
### Reset  
メッセージや実行中の変数をリセットします。  
たとえば以下のコードが存在します。  
  
```python
print(x)
x = 1
```  
  
・1行目を「run line」を実行するとNameErrorが発生します。  
・2行目を実行した後に1行目を実行すると「1」と表示されます。  
　これは、内部的には変数の内容を覚え続けているためです。  
・この後は、「Reset」を実行するまでは、1行目を「run line」してもNameErrorになりません。  
・「Reset」を実行することで初期状態にもどり1行目を「run line」するとNameErrorになります。  
  
## ツールバー  
![image.png](/image/638e4275-1767-3b54-afa3-889b68caf7bc.png)  
  
### スクリーンショットを撮る（CTRL+SHIFT+2）  
全てのスクリーンが以下のようなマスクがかかり範囲を選択できるようになります。  
![image.png](/image/b34908cd-3778-bade-4345-ac5c505d3462.png)  
  
範囲を選択した箇所は画像として保存されて、IDEに表示されます。  
ファイル名は環境設定により、手動で入力するか、自動で設定するかを選択可能です。  
  
#### Pattern  
「スクリーンショットを撮る」で作成した画像はPatternオブジェクトに変換可能です。  
IDE上の画像をクリックすると下記の画面が表示されます。  
  
![image.png](/image/0f997740-9b38-d00c-68d3-66b98bfa3fc6.png)  
  
・ファイルタブはファイルのパス情報が記載されています。  
・マッチングプレビューは類似度を設定できます。  
・ターゲットオフセットは画像のどこをターゲットにするかを指定できます。  
  
以下の画像をマッチングプレビューを使用して類似度を変更してみます。  
![image.png](/image/c9fc2b82-e2bd-3c47-669c-61651cd6ecba.png)  
  
0.7の場合は類似の画像が2つ検知されます。  
![image.png](/image/4f0aff9b-abc8-9e9a-93ce-a02f9cc00078.png)  
  
0.98にあげると1つのみとなります。  
![image.png](/image/f354ac41-ff91-0b18-739a-509854a5ec65.png)  
  
このように類似度を実際の画面を確認しながら選択することができます。  
パターン設定画面で設定したファイルは以下のようにPatternオブジェクトとして扱われるようになります。  
  
```
pattern = Pattern("image.png").similar(0.98).targetOffset(-153,-87)
```  
  
Patternの公式ドキュメント  
https://sikulix-2014.readthedocs.io/en/latest/pattern.html?highlight=Pattern  
  
コード  
https://github.com/RaiMan/SikuliX1/blob/08eb817c70afafdabff87d46cadb372255a66ad7/API/src/main/java/org/sikuli/script/Pattern.java  
  
### 画像を挿入する  
ディスク上にある画像を選択してIDEに表示します。  
選択した画像はsikuliフォルダにコピーされて使用されます。  
作成後の画像については「スクリーンショットを撮る」と同じくPatternとして処理できます。  
  
### Region  
スクリーン上の範囲を選択できます。  
IDE上は以下のように表示されます。  
![image.png](/image/554a6874-fa8c-4d21-f574-b1745794837c.png)  
※左下の赤い領域が選択した箇所  
  
IDE上は先ほどの「スクリーンショットを撮る」と「画像を挿入する」と同じような見え方をしますが、実際のコード上は異なります。  
メニューの「表示」→「Toggle Thumnails」のチェックをOFFにすると実際のコードが表示されます。  
  
![image.png](/image/b2534826-a926-9938-a3a5-cac504707874.png)  
  
RegionのfindメソッドやfindTextメソッドを用いて領域範囲内の画像やテキストの検索を行います。  
  
Regionについての公式ドキュメントは下記の通りです。  
https://sikulix-2014.readthedocs.io/en/latest/region.html  
  
コードは以下になります。  
https://github.com/RaiMan/SikuliX1/blob/08eb817c70afafdabff87d46cadb372255a66ad7/API/src/main/java/org/sikuli/script/Region.java  
  
### Location  
Locationはスクリーン上の一点の位置（x、y）を取り扱います。  
LocationをクリックするとRegionと同様にスクリーン上の範囲を選択できるようになります。  
  
Locationの座標として採用されるのは赤い十字の中心となります。  
![image.png](/image/cb76896f-017e-8dc7-ebba-e534a77eca35.png)  
  
作成したLocationについては以下のように使用します。  
  
```
# 674,217 に移動してクリックする
Location(674, 217).click()
```  
  
Locationについての公式ドキュメントは下記の通りです。  
https://sikulix-2014.readthedocs.io/en/latest/location.html  
  
またソースコードは以下になります。  
https://github.com/RaiMan/SikuliX1/blob/08eb817c70afafdabff87d46cadb372255a66ad7/API/src/main/java/org/sikuli/script/Location.java  
  
### Offset  
Locationと同様に選択を行います。  
このときに作成されるコードは以下のようになります。  
  
```
offset = Offset(58, 28)
```  
  
Offsetの目的については以下に説明が記載されています。  
https://answers.launchpad.net/sikuli/+question/446476  
  
たとえば以下のような画面があり、「含まれるテキスト」項目に入力をする必要がある場合を考えましょう。  
![image.png](/image/41cf0193-0af2-521d-425b-8a0244f80118.png)  
  
この場合、![image.png](/image/9c8b98ce-3a71-3435-a8fe-69fed467698f.png)という画像から下に一定量はなれた箇所をクリックする必要があります。  
この際にOffsetを使用します。  
  
実際のコードにすると以下のようになります。  
![image.png](/image/78f86726-3636-af20-05cd-5d52e6524c27.png)  
  
Offsetのコードは以下になります。  
https://github.com/RaiMan/SikuliX1/blob/08eb817c70afafdabff87d46cadb372255a66ad7/API/src/main/java/org/sikuli/script/Offset.java  
  
  
### Show  
カーソルのある行のRegion,ファイル名,Patternの画像がスクリーン全体のどこにあるかを検索して強調表示します。  
  
![image.png](/image/0be6791d-2a81-1fe0-7145-8f1bdab91e83.png)  
  
### ShowIn  
基本的にShowと同じ使い方です。  
Showは検索範囲がスクリーン全体ですが、ShowInの場合、検索前に検索範囲を矩形で設定できます。  
  
# 代表的な処理  
## 画像検索  
openCvを利用して画像の検索が行えます。  
  
### find()  
指定の範囲で指定の画像が存在するか検索します。存在する場合は最初に見つけた画像の位置を返却します。  
見つからない場合はFindFailed例外が発生します。  
  
```python
rc = Region(8,-836,876,597)
try:
    ret = rc.find(Pattern("masktest1.png"))
except FindFailed:
    print ('not found')
    exit()
ret.highlight(2)
exit()

```  
  
### wait()  
指定の範囲で指定の画像が出現するまで待機します。出現した場合は最初に見つけた画像の位置を返却します。  
見つからない場合はFindFailed例外が発生します。  
  
```python
rc = Region(8,-836,876,597)
try:
    ret = rc.wait(Pattern("masktest1.png"), 10)
except FindFailed:
    print ('not found')
    exit()
ret.highlight(2)
exit()

```  
  
### exists()  
指定の画像が出現するまで待機します。存在した場合は画像領域を返します。  
もし見つからない場合、Noneを返します。  
  
```python
rc = Region(8,-836,876,597)

ret = rc.exists(Pattern("masktest1.png"), 10)
print(ret)
if ret is None:
    print ('Not found')
    exit()
ret.highlight(2)
```  
  
### has()  
指定の画像が存在するか確認します。存在した場合はTrueを返します。  
もし見つからない場合、Falseを返します。  
  
```python
rc = Region(8,-836,876,597)

ret = rc.has(Pattern("masktest1.png"), 10)
print(ret)
```  
  
#### waitVanish  
指定の範囲で指定の画像が消えるまで待ちます。指定秒の間に消えなかったらFalse、消えたらTrueを返します  
  
```python
rc = Region(8,-836,876,597)
ret = rc.waitVanish(Pattern("masktest1.png"), 10)
print(ret)

```  
  
### 変更点の監視  
onChangeに変更が発生した場合に処理するイベントハンドラを登録します。  
  
```python
# https://sikulix-2014.readthedocs.io/en/latest/region.html?highlight=observe%20
def changed(event):
        print "something changed in ", event.region
        for ch in event.getChanges():
                ch.highlight() # highlight all changes
        wait(1)
        for ch in event.getChanges():
                ch.highlight() # turn off the highlights

# 範囲選択が表示されてRegionを選択する
r = selectRegion("select a region to observe")

# 50ピクセル変更したらchangedを呼び出す

r.onChange(50, changed)

# 30秒間監視
r.observeInBackground(); wait(30)
r.stopObserver()
```  
  
この例では、チェックボックスを付けたりして画面が変更した箇所が強調表示されます。  
![image.png](/image/6efcf29f-304b-85df-6e67-e8d0c90f6f04.png)  
  
  
  
### findAll()  
指定範囲内で指定の画像が存在する箇所を全て列挙します。存在しない場合は空配列となります。  
  
```python
rc = Region(8,-836,876,597)

list = rc.findAll(Pattern("masktest1.png"))
for i in list:
    i.highlight(2)
    print(i.getScore())
    
exit()
```  
  
### マスクと透過画像の使用  
画像検索では透過画像またはマスクを利用して画像の一部を無視して検索が行えます。  
  
検索範囲：  
![image.png](/image/6aa2228a-ebe1-e843-fb03-14d9c250d17b.png)  
  
#### 透過画像を利用する例  
検索画像：  
![image.png](/image/fadcf80e-7b3d-8923-65a0-96af6015ffee.png)  
  
コード：  
![image.png](/image/0def0272-5b22-393f-daba-426a01785cd8.png)  
  
findAllでマッチした画像を検索して、そのスコアを表示します。  
この例だと以下のようになります。  
  
![image.png](/image/6011cc6d-f48f-d542-1bd0-e50e4ac61b02.png)  
  
#### 黒をマスクとして利用する  
検索画像  
![image.png](/image/f6a4cf06-9daa-9c33-8b42-92b87014b976.png)  
  
  
コード：  
![image.png](/image/51c77b15-c36c-75dc-c966-5f443888ee5b.png)  
  
今回は「Pattern("masktest_black.png").exact().mask()」と「exact()」を利用して類似度を0.99まで上げています。  
この例での結果は透過の場合と同じで2件、類似度が1.0で取得できました。  
  
#### マスク画像の合成  
検索画像１　masktest1.png  
![image.png](/image/d8a1f419-eb8e-2598-c8c7-d415ac150ef6.png)  
  
マスク画像　masktest1mask.png  
![image.png](/image/7b22a33b-1375-edaf-7bd6-f9ebe8144dd0.png)  
  
コード  
  
```python
rc = Region(8,-836,876,597)

list = rc.findAll(Pattern("masktest1.png").exact().mask("masktest1mask.png"))
for i in list:
    i.highlight(2)
    print(i.getScore())
    
exit()

```  
  
結果については「黒をマスクとして利用する」場合と同じになります。  
  
## テキスト検索  
sikulix1.1.4では[Tess4J](http://tess4j.sourceforge.net/)を使用してテキストの認識をします。  
  
ただし、OCRなので100%の精度を期待してはいけないです。  
  
### 使用例  
(1)言語情報を取得します。  
https://github.com/tesseract-ocr/tessdata/tree/3.04.00  
日本語の場合は「jpn.traineddata」になります。  
  
(2)下記のフォルダに配置します。  
C:\Users\ユーザ名\AppData\Roaming\Sikulix\SikulixTesseract\tessdata  
  
(3)特定の範囲内のテキストを読み込んでみます  
  
```python
tr = TextOCR.start()
tr.setLanguage("jpn")
r = Region(0,63,159,74)
print r.text()
```  
  
### findText()  
指定の文字が存在するかを検査します。存在した場合は文字の領域のRegionを返します。  
もし見つからない場合、FindFailed例外が発生します。  
  
```python
import sys
reload(sys)
sys.setdefaultencoding('utf-8')

Settings.OcrLanguage = 'jpn'
#Settings.OcrTextSearch = False   # バージョンによってTrueにするとImage.isValidでNullPointerError
Settings.OcrTextRead  = True
Settings.SwitchToText = False
TextOCR.start()
r = Region(0,0,100,150)

try:
    ret = r.findText(u'あい')
except FindFailed as ex:
    print ('error' + str(ex))
    exit(1)
ret.highlight(2)

```  
  
### waitText()  
指定の文字が出現するまで待機します。存在した場合は文字の領域のRegionを返します。  
もし指定した秒数まで見つからない場合はFindFailed例外が発生します。  
  
```python
import sys
reload(sys)
sys.setdefaultencoding('utf-8')

Settings.OcrLanguage = 'jpn'
#Settings.OcrTextSearch = False   # バージョンによってTrueにするとImage.isValidでNullPointerError
Settings.OcrTextRead  = True
Settings.SwitchToText = False
TextOCR.start()
# tr = TextOCR.start()
# tr.setLanguage('jpn')
r = Region(0,0,100,150)

try:
    ret = r.waitText(u'あい', 10) # 10秒待機
except FindFailed as ex:
    print ('error' + str(ex))
    exit(1)
ret.highlight(2)
```  
  
### hasText()  
指定の文字が存在するかを検査します。存在した場合は文字の領域のRegionを返します。  
もし見つからない場合、Noneを返します。  
  
```python
import sys
reload(sys)
sys.setdefaultencoding('utf-8')

Settings.OcrLanguage = 'jpn'
#Settings.OcrTextSearch = False   # バージョンによってTrueにするとImage.isValidでNullPointerError
Settings.OcrTextRead  = True
Settings.SwitchToText = False
TextOCR.start()
r = Region(0,0,100,150)
ret = r.hasText(u'あい')
ret.highlight(2)
exit()
```  
#### existsText()  
指定の文字が出現するまで待機します。存在した場合は文字の領域のRegionを返します。  
もし見つからない場合、Noneを返します。  
  
```python
import sys
reload(sys)
sys.setdefaultencoding('utf-8')

Settings.OcrLanguage = 'jpn'
#Settings.OcrTextSearch = False   # バージョンによってTrueにするとImage.isValidでNullPointerError
Settings.OcrTextRead  = True
Settings.SwitchToText = False
TextOCR.start()
r = Region(0,0,100,150)
ret = r.existsText(u'あい',10)
if ret is None:
    print ('Not found')
    exit()
ret.highlight(2)
exit()
```  
### findAllText()  
指定範囲内で指定の文字が存在する箇所を全て列挙します。存在しない場合は空配列となります。  
  
  
```python
Settings.OcrLanguage = 'jpn'
#Settings.OcrTextSearch = False   # バージョンによってTrueにするとImage.isValidでNullPointerError
Settings.OcrTextRead  = True
Settings.SwitchToText = False
TextOCR.start()
r = Region(0,0,100,150)
result = r.findAllText(u'あい')
for item in result:
    item.highlight(2)
```  
  
### findWord/findLine  
単語単位または行単位で検索します。単語間に空白を開ける英語だと予期したとおりに動かしやすいですが、日本語のような言語だと厳しいかもしれません。  
見つからない場合は、「java.lang.ClassCastException」が発生してますが、おそらく、意図していない例外だと思います。  
  
```python
import sys
# 対策:public org.python.core.PyObject o object has no attribute setdefaultencoding 
reload(sys) 
sys.setdefaultencoding('utf-8')

Settings.OcrLanguage = 'jpn'
#Settings.OcrTextSearch = False   # TrueにするとImage.isValidでNullPointerError
# Settings.OcrTextRead  = True
# Settings.SwitchToText = False
TextOCR.start()
r = Region(0,2,243,203)

ret = r.findWord(u'He')
ret.highlight(2)
ret = r.findLine(u'He')
ret.highlight(2)

# 「彼」の前後にスペースがないと認識しないかも
ret = r.findWord(u'彼')
ret.highlight(2)

# 「彼」の前後にスペースがなくてもOK
ret = r.findLine(u'彼')
ret.highlight(2)
exit(1)

```  
findWord(u'He')  
![image.png](/image/93e903b3-a71c-dbef-9c47-9a70e084b731.png)  
  
findLine(u'He')  
![image.png](/image/cab7bad9-5a07-70c4-d488-b38a29aeda0c.png)  
  
findWord(u'彼')  
![image.png](/image/3bcb4723-57f0-91ed-29ce-6ca8fb5b64b9.png)  
  
findLine(u'彼')  
![image.png](/image/a9d93420-e4e0-5e01-c0f0-5aa10e62a497.png)  
  
## 画面操作  
  
### click(PSMRL[, modifiers])  
指定の座標でクリックを行います。  
PSMRLにはPattern、string,match, region, locationが指定できます。  
modifiersは省略可能な引数で１つ以上のキーボード操作を指定できます。  
  
```python
# find()やwait()で最期に見つけた座標をクリックする
click() 

# 座標を指定してクリックする
click(Location(103, 635))

# 指定の画像をクリックする
click("test.png")

# 指定の画像をSHIFTを押しながらクリックする
click("test.png", KeyModifier.SHIFT)
```  
  
### mouseMove(PSRML)  
指定の座標にマウスポインタを移動させます。  
PSMRLにはPattern、string,match, region, locationが指定できます。  
   
```python
# 指定の画像へ移動
mouseMove("test.png")

# 指定の座標に移動
mouseMove(Location(278, 182))

```  
  
  
### doubleClick(PSMRL[, modifiers])  
指定の座標でダブルクリックをします。  
PSMRLにはPattern、string,match, region, locationが指定できます。  
modifiersは省略可能な引数で１つ以上のキーボード操作を指定できます。  
  
```python

# find()やwait()で最期に見つけた座標をダブルクリックする
doubleClick()

# 座標を指定してダブルクリックする
doubleClick(Location(103, 635))

# 指定の画像をダブルクリックする
doubleClick("test.png")
```  
  
### rightClick(PSMRL[, modifiers])  
指定の座標で右クリックをします。  
PSMRLにはPattern、string,match, region, locationが指定できます。  
modifiersは省略可能な引数で１つ以上のキーボード操作を指定できます。  
  
```python
rightClick("test.png")
sleep(5) # コンテキストメニューが表示される。実運用ではwaitを使う
```  
  
### dragDrop(PSMRL, PSMRL[, modifiers])  
ドラッグアンドドロップを実行します。  
PSMRLにはPattern、string,match, region, locationが指定できます。  
modifiersは省略可能な引数で１つ以上のキーボード操作を指定できます。  
  
操作対象の領域  
![image.png](/image/9c36548f-4259-2ac1-5124-dfe7fb156f10.png)  
  
  
コード  
![image.png](/image/9629e3d1-5dd6-99e5-bdea-1cfbe8b2dbe1.png)  
  
実行結果  
![image.png](/image/22f0a163-4a73-cff7-de15-5d33adf9ea33.png)  
  
※あくまでテストです。実際にエクスプローラーの操作を画像認識でやると苦労します。  
たとえば、フォルダは以下のように中にファイルがあるかどうかでアイコンが変わったりします。  
  
![image.png](/image/51ea88b1-84e9-4643-22ab-6ddf939c52db.png)  
  
  
### type([PSMRL, ]text[, modifiers])  
PSMRLにはPattern、string,match, region, locationが指定できます。  
textには入力するASCIIの文字を指定します。  
modifiersには同時に押下するSHIFTなどのキーを指定します。  
  
```python
# CTRL+Aを押して全選択する
type(Location(351, 122), 'a' , KeyModifier.CTRL)
# 選択している文字を削除
type(Key.DELETE)
# abcをタイプ
type(Location(351, 122), 'abc')

```  
  
### paste([PSMRL, ]text)  
クリップボード経由で任意のテキストを貼り付けます。日本語文字をテキストに入力する場合に使用します。  
PSMRLにはPattern、string,match, region, locationが指定できます。  
textには貼り付けたい文字を指定します。  
  
コード：  
![image.png](/image/d2857593-8e2c-d255-3728-62b71da26805.png)  
  
実行前：  
![image.png](/image/9dfdfc66-31df-d8a9-0076-a90d69eebc05.png)  
  
実行後：  
![image.png](/image/dc2fdbfe-198d-8e5a-68e0-ae56103dea06.png)  
  
  
### その他操作、低レベルの画面操作  
以下を参照してください。  
https://sikulix-2014.readthedocs.io/en/latest/region.html#low-level-mouse-and-keyboard-actions  
  
# プライマリー以外のスクリーンの操作  
## マルチディスプレイでの操作  
以下のようなマルチディスプレイ環境だとします。  
![image.png](/image/69b13b08-ef26-397c-8443-6acc47936d9c.png)  
  
・getNumberScreens()コマンドでディスプレイが幾つあるか取得します。  
・３のディスプレイを操作する場合は「Screen(3).click(...)」と記述します。  
・座標の関係は以下のようになります。（[参照](https://sikulix-2014.readthedocs.io/en/latest/screen.html?highlight=Screen#multi-monitor-environments))  
![image.png](/image/63a92942-4bb8-e1ac-33e3-12ea72aeb724.png)  
  
マルチディスプレイ環境での実装例：  
![image.png](/image/20139621-f47d-1672-6ae4-06186e1c0caa.png)  
この例では、全てのスクリーンに対して指定の画像が存在するかを調べて、存在すれば操作をするような動きになります。  
  
## VNC経由の操作  
TigerVNC Viewerをベースに実装したVNCの処理を経由して別PCの画面が操作可能です。  
以下の例ではVNC経由で画面をキャプチャしたものになります。  
  
```python
vnc = vncStart('192.168.0.18', port=5900, password=u"パスワード")
print(vnc)
tmp = vnc.capture(Region(0,0,1000,1000))
path = tmp.save()
print(path)
vnc.stop()
exit(1)
```  
  
## Androidに対する操作  
Sikulixを用いてAndroidを操作するにはAndroidStudioをインストールした際に追加されるADBが必要です。  
ADBを経由してAndroidのタップやスクリーンキャプチャを実現してます。  
  
実験段階のためか、不安定な動きです。  
  
### IDEの起動方法  
ADBのパスをSikulixに教える必要があるので、環境変数sikulixadbにADBのパスを指定して、SikulixIDEを起動します。  
  
```
@echo off
SET JAVA_HOME=C:\pleiades201904\java\12
set sikulixadb=C:\Program Files (x86)\Android\android-sdk\platform-tools\adb.exe
"%JAVA_HOME%\bin\java.exe"  -jar C:\tool\sikulix\sikulixide-1.1.4-SNAPSHOT-complete.jar
```  
  
  
### Androidの画像キャプチャの方法  
（１）AndroidをUSBでつなげて「menuToolAndroid」を選択します  
  
（２）ポップアップが表示されるので「Default Android」を選択します。  
![image.png](/image/7866e4b1-be56-386c-c397-dafd9a138b5a.png)  
  
  
（３）「スクリーンショットを撮る」を実行するとAndroidからキャプチャを撮るか聞かれるので「はい」を選択します  
![image.png](/image/26910984-26d7-49fe-5a8b-f565b01f1aae.png)  
  
（４）PC上にAndroidのキャプチャが表示されるので範囲を選択します。  
![image.png](/image/563f9649-1758-92bb-fd1d-09342482f209.png)  
  
（５）IDEに選択範囲が表示されます。あとはPCと同じです。  
![image.png](/image/b9563db1-c086-0b83-3c90-13e5223c8fd3.png)  
  
  
#### （４）でフリーズする場合  
Xperia SO-02Kの実機だと固まりました。  
この場合、ADBDevice.javaのcaptureDeviceScreenMatを以下のように修正してむりやり動かします。  
  
**ADBDevice.java**  
```java:ADBDevice.java
  public Mat captureDeviceScreenMat(int x, int y, int actW, int actH) {
// 略
            if (!endOfStream) {
              if (pixel[3] == -1) {
                if (npr >= y && npc >= x && npc < maxC) {
                  image[atImage++] = pixel[0];
                  image[atImage++] = pixel[1];
                  image[atImage++] = pixel[2];
                  image[atImage++] = pixel[3];
                }
              } else {
                log(-1, "buffer problem: %d", nPixels);
                // TODO Xperia SO-02K pixel[3] is 0
                //return null;
              }
            } else {
              break;
            }
```  
  
無理やり動かしたせいか、PC上の範囲選択の座標がずれて上手くいかなくなっていました。  
  
  
### 実行方法  
ADBScreen.start()で作成したオブジェクトに対して操作を行うことで、Androidの操作が行えます。  
  
![image.png](/image/aa6e64c5-2737-831f-a5f3-7879b3ddcc38.png)  
  
  
## サーバー経由の実行  
HTTPサーバーを起動して、外部からの自動操作を受け付けます。  
  
(1)sikulixをサーバーとして起動します。  
  
```text
%JAVA_HOME%\bin\java.exe  -jar sikulix.jar -s
```  
  
(2)ブラウザで以下を入力してスクリプトが格納されているフォルダを設定します。  
  
```
http://127.0.0.1:50001/scripts/c:/tool/sikulix
```  
  
操作対象の「.sikuli」を格納しているフォルダをscriptsの後に入力してください。  
上記の例の場合だと「c:\tool\sikulix」が選択されます。  
  
(3)「hello.sikuli」スクリプトを実行します。  
  
```
http://127.0.0.1:50001/run/hello
```  
  
  
その他は以下のページ参照してください  
https://sikulix-2014.readthedocs.io/en/latest/scenarios.html#experimental-runserver-run-scripts-from-anywhere-with-zero-delay  
  
なお、1.1.4の2019年4月リリース版だと動作しなかったのでGithubからビルドした資材を使用して確認しました。  
  
## PythonServer経由の実行  
[py4J](https://www.py4j.org/index.html)を使用してJythonではなく、本物のpythonで実装ができます。この機能は現在開発中です。  
  
(1)PythonServerを起動  
  
```
%JAVA_HOME%\bin\java.exe  -jar sikulix.jar -p
```  
  
(2)py4Jのインストール  
  
```
pip install py4j
```  
  
(3)sikulix4pythonの入手  
https://github.com/RaiMan/sikulix4python  
  
(4)Pythonのコードを記述して実行  
  
```python
from sikulix4python import *
switchApp("notepad") # メモ帳にフォーカスが遷る
```  
  
# その他ドキュメントがおいついてなさそうなところ  
## HTTP上の画像の使用  
以下によると「addHTTPImagePath」でサイトを追加して使用するような記述になっています。  
https://sikulix-2014.readthedocs.io/en/latest/scripting.html  
  
1.1.4ではaddHTTPImagePathは廃止されており、以下のようにして実現できます。  
  
```python
# ImagePath.addHTTP('https://pbs.twimg.com/profile_images/1111729635610382336/')
# addHTTPImagePathは以下に1.1.4統合されたっぽい。
addImagePath('https://pbs.twimg.com/profile_images/1111729635610382336/')
click('_65QFl7B_200x200.png')

```  
  
## PowerShellの実行  
下記のドキュメントで「returnCode = runScript('powershell get-process')」のような記載ができるようにかかれていますが、1.1.4ではps1という拡張子を持つ外部ファイルを実行することでのみPowerShellを使用できます。  
https://sikulix-2014.readthedocs.io/en/latest/scripting.html  
  
以下の例ではPowerShellの標準出力をUTF-8にして無理やり日本語を扱えるようにしています。  
  
**test.ps1**  
```powershell:test.ps1
chcp 65001
ls
```  
  
  
```python
import sys
reload(sys)
sys.setdefaultencoding('utf-8')
returnCode = runScript('C:\\tool\\sikulix\\test.ps1')
print returnCode
print RunTime.get().getLastCommandResult().encode('utf-8')
print RunTime.get().getLastCommandResult().encode('utf-8').replace('ディレクトリ', 'フォルダ')
```  
  
このコードはlsの結果で表示される「ディレクトリ」という文字列をpythonの処理で「フォルダ」に変換してコンソールに出力しています。  
  
**出力**  
```text:出力
ディレクトリ: C:\tool\sikulix 
Mode LastWriteTime Length Name 
---- ------------- ------ ---- 
d----- 2019/06/03 16:41 hello2.sikuli 
// 略
Active code page: 65001 
フォルダ: C:\tool\sikulix 
Mode LastWriteTime Length Name 
---- ------------- ------ ---- 
```  
  
