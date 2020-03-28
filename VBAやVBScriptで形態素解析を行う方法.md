このドキュメントではVBAやVBScriptを使用して形態素解析を行う方法について説明します。  
  
単語の分割だけでいい場合は下記を参考。  
  
 __VBScriptで分かち書きを実行（MS標準機能のみで実装）__   
http://qiita.com/nezuq/items/2e4e0cc63316474b630d  
  
# NMeCab  
NMeCabは形態素解析エンジンMeCabの解析処理部分を、.NETライブラリとして移植したものであり、下記からダウンロードできます。  
  
http://sourceforge.jp/projects/nmecab/releases/p11745  
  
今回は、NMecabをCOMでラッパーするNMecabComを作成し、そこを経由してVBA,VBScriptから形態素解析を行います。  
  
![b0232065_21104163.png](/image/df53d91d-4ed3-e1b1-20a0-9548afd4389c.png)  
  
具体的な作成方法については下記を参考の事。  
[VBAまたはVBSからCOM経由で使用できる.NETのライブラリの作成方法](http://qiita.com/mima_ita/items/efcd1a6ea86f09047984 "VBAまたはVBSからCOM経由で使用できる.NETのライブラリの作成方法")  
  
  
作成したNMeCabComは下記からダウンロードできます。  
http://needtec.sakura.ne.jp/release/NMeCabCom.zip  
  
  
# 環境  
・Visual Studio2008 Pro  
・.NET Framework 2.0 を対象とする。  
・32ビットのOffice2010  
  
  
# 使用方法  
## インストール方法  
１．下記からファイルをダウンロードします  
http://needtec.sakura.ne.jp/release/NMeCabCom.zip  
  
２．NMeCabCom\binにあるsetup.batを管理者権限で実行してください。  
  
もし、Excelの64ビットのプロセスで使用したい場合は、NMeCabCom.dllとLibNMeCab.dllを64ビットでビルドしなおしてください。  
  
また、アンインストールを行い対場合はuninstall.batを管理者権限で実行してください。  
  
## Excel VBAからの使用方法  
インストールが適切に終了していれば、参照設定にNMecabComが追加されるのでチェックを入れてください。  
  
![1w04banj.png](/image/f501ff58-5826-38db-e82f-03270ac5298f.png)  
  
その後以下のような実装をします。  
  
```vb

    Dim t As New NmcTagger
    Dim c As NmcNodeCollection
    Dim p As New NmcParam
    p.dicdir = "C:\back\NMeCabCom\bin\ipadic"
    Call t.Create(p)
    Set c = t.Parse("私の名前はLです")
    Dim i As Long
    For i = 0 To c.Count - 1
        Debug.Print c.GetItem(i).Surface & " " & c.GetItem(i).Feature
    Next
```  
  
## 作成例  
  
  
NmcTaggerに渡すNmcParamにはシステム辞書とユーザ辞書を指定できます。  
ユーザー辞書の作成については、[本家のMecab](http://mecab.googlecode.com/svn/trunk/mecab/doc/index.html "本家のMecab")のホームページを参考に作成してください。  
  
## VBScriptからの使用方法  
  
CreateObjectで遅延バインディングする場合は以下のようになります。  
  
```vb

dim t
dim ret
dim i
dim p
dim node
set t = CreateObject("NMeCabCom.NmcTagger")
WScript.Echo TypeName(t)

set p = CreateObject("NMeCabCom.NmcParam")

p.dicdir = "./ipadic"
p.userdic = ""
t.Create(p)

set ret = t.Parse("This is a pen.")
WScript.Echo TypeName(ret)

For i = 0 To ret.Count- 1
  WScript.Echo TypeName(ret.GetItem(i))
  WScript.Echo ret.GetItem(i).Surface
Next
```  
  
## 応用例  
bin/NicoMsgAnalyze.xlsm はニコニコ動画のコメントを集計して解析するサンプルです。  
  
 __ExcelVBA を使ったニコニコ動画のコメントの分析__   
http://www.nicovideo.jp/watch/sm22332042  
