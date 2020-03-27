## 目的  
Cabocha0.68をインストールしてPythonで係受け解析を行う  
  
## 前提  
Mecabをインストールしてあること  
https://code.google.com/p/mecab/downloads/list  
  
ここではmecab-0.996.exeをUTF-8でインストールしてあるものとする。  
  
## Cabochaのインストール  
1.cabocha-0.68.exeのダウンロード  
　http://code.google.com/p/cabocha/downloads/list  
  
2.ダウンロードしたEXEの実行。この際、選択する文字コードはMecabの文字コードと同一にする。  
※ここではUTF-8を選択  
  
3.環境変数のパスに”C:\Program Files (x86)\CaboCha\bin”を通してcabochaを実行できるようにしておく  
れはpythonがdllにアクセスするのにも必要である。  
  
4.実行確認を行う  
input.txtというUTF8のファイルを作成して解析したい文字列を入力して、コマンドプロンプトから下記を実行する。  
  
```
cabocha < input.txt > out.txt
```  
  
  
  
適切に解析できれば下記のようなファイルが出力される。  
  
```

                ここは---D
                まりさの-D
    ゆっくりプレイスだよ！
EOS
```  
  
なおここでファイルを経由しているのはコマンドプロンプトでUTF-8を扱えないためである。  
  
  
```
ここはまりさのゆっくりプレイスだよ！
EOS
```  
こうなってしまった場合、input.txtの文字コードがutf-8になっていない場合がある。  
（メモ帳で作成した場合、デフォルトがANSIなので注意）  
  
  
また、以下のようなエラーが発生する場合がある。  
  
```
svm.cpp(140) [version == MODEL_VERSION] incompatible version: 101
svm.cpp(751) [size >= 2] dep.cpp(79) [!failed] no such file or directory: C:\Program Files (x86)\CaboCha\etc\..\model\dep.ipa.model
```  
  
この場合は、cabochaのバージョンアップが適切に行えていないので、下記のフォルダを削除すること。  
  
```
C:\Users\ユーザ名\AppData\Local\VirtualStore\Program Files(x86)\CaboCha
```  
  
## PythonからCabochaを使用できるようにする。  
1.cabocha-0.68.tar.bzのダウンロード  
　http://code.google.com/p/cabocha/downloads/list  
このファイルはLhaplusなどで解凍できる  
  
2.解凍したフォルダ中のpythonフォルダにカレントディレクトリを移動させて下記のコマンドを実行する。  
  
```
python setup.py install
```  
  
3.以下のエラーが発生する。  
  
```
Traceback (most recent call last):
  File "setup.py", line 13, in <module>
    version = cmd1("cabocha-config --version"),
  File "setup.py", line 7, in cmd1
    return os.popen(str).readlines()[0][:-1]
IndexError: list index out of range
```  
  
これは、Windowsにはcabocha-configがインストールされていないために発生する  
  
4.setup.pyを変更する。  
  
```py:変更前
# !/usr/bin/env python

from distutils.core import setup,Extension,os
import string

def cmd1(str):
    return os.popen(str).readlines()[0][:-1]

def cmd2(str):
    return string.split (cmd1(str))

setup(name = "cabocha-python",
	version = cmd1("cabocha-config --version"),
	py_modules=["CaboCha"],
	ext_modules = [
		Extension("_CaboCha",
			["CaboCha_wrap.cxx",],
			include_dirs=cmd2("cabocha-config --inc-dir"),
			library_dirs=cmd2("cabocha-config --libs-only-L"),
			libraries=cmd2("cabocha-config --libs-only-l"))
			])


```  
  
version と、ext_modulesの内容をインストールした情報に書き換える。  
  
```py:変更後
# !/usr/bin/env python

from distutils.core import setup,Extension,os
import string

def cmd1(str):
    return os.popen(str).readlines()[0][:-1]

def cmd2(str):
    return string.split (cmd1(str))

setup(name = "cabocha-python",
	version = "0.68",
	py_modules=["CaboCha"],
	ext_modules = [
		Extension("_CaboCha",
			["CaboCha_wrap.cxx",],
			include_dirs=[r"C:\Program Files (x86)\CaboCha\sdk"],
			library_dirs=[r"C:\Program Files (x86)\CaboCha\sdk"],
			libraries=['libcabocha'])
])
```  
  
5.再度、setup.pyの実行  
  
```
python setup.py install
```  
  
6.以下のサンプルプログラムを入力して試す。  
  
```py
# !/usr/bin/python
# -*- coding: utf-8 -*-

import CaboCha

# c = CaboCha.Parser("");
c = CaboCha.Parser("")

sentence = "帽子を返す"

# print c.parseToString(sentence)

# tree =  c.parse(sentence)
#
tree =  c.parse(sentence)
print tree.toString(CaboCha.FORMAT_TREE)
print tree.toString(CaboCha.FORMAT_LATTICE)
# print tree.toString(CaboCha.FORMAT_XML)

for i in range(tree.chunk_size()):
    chunk = tree.chunk(i)
    print 'Chunk:', i
    print ' Score:', chunk.score
    print ' Link:', chunk.link
    print ' Size:', chunk.token_size
    print ' Pos:', chunk.token_pos
    print ' Head:', chunk.head_pos # 主辞
    print ' Func:', chunk.func_pos # 機能語
    print ' Features:',
    for j in range(chunk.feature_list_size):
        print '  ' + chunk.feature_list(j) 
    print
    print 'Text' 
    for ix  in range(chunk.token_pos,chunk.token_pos + chunk.token_size):
      print ' ', tree.token(ix).surface 
    print

for i in range(tree.token_size()):
    token = tree.token(i)
    print 'Surface:', token.surface
    print ' Normalized:', token.normalized_surface
    print ' Feature:', token.feature
    print ' NE:', token.ne # 固有表現
    print ' Info:', token.additional_info
    print ' Chunk:', token.chunk
    print

```  
  
```サンプルの出力結果
帽子を-D
    返す
EOS

* 0 1D 0/1 0.000000
帽子	名詞,一般,*,*,*,*,帽子,ボウシ,ボーシ
を	助詞,格助詞,一般,*,*,*,を,ヲ,ヲ
* 1 -1D 0/0 0.000000
返す	動詞,自立,*,*,五段・サ行,基本形,返す,カエス,カエス
EOS

Chunk: 0
 Score: 0.0
 Link: 1
 Size: 2
 Pos: 0
 Head: 0
 Func: 1
 Features:   FCASE:を
  FHS:帽子
  FHP0:名詞
  FHP1:一般
  FFS:を
  FFP0:助詞
  FFP1:格助詞
  FFP2:一般
  FLS:帽子
  FLP0:名詞
  FLP1:一般
  FRS:を
  FRP0:助詞
  FRP1:格助詞
  FRP2:一般
  LF:を
  RL:帽子
  RH:帽子
  RF:を
  FBOS:1
  GCASE:を
  A:を

Text
  帽子
  を

Chunk: 1
 Score: 0.0
 Link: -1
 Size: 1
 Pos: 2
 Head: 0
 Func: 0
 Features:   FHS:返す
  FHP0:動詞
  FHP1:自立
  FHF:基本形
  FFS:返す
  FFP0:動詞
  FFP1:自立
  FFF:基本形
  FLS:返す
  FLP0:動詞
  FLP1:自立
  FLF:基本形
  FRS:返す
  FRP0:動詞
  FRP1:自立
  FRF:基本形
  LF:返す
  RL:返す
  RH:返す
  RF:返す
  FEOS:1
  A:基本形

Text
  返す

Surface: 帽子
 Normalized: 帽子
 Feature: 名詞,一般,*,*,*,*,帽子,ボウシ,ボーシ
 NE: None
 Info: None
 Chunk: <CaboCha.Chunk; proxy of <Swig Object of type 'CaboCha::Chunk *' at 0x0274A170> >

Surface: を
 Normalized: を
 Feature: 助詞,格助詞,一般,*,*,*,を,ヲ,ヲ
 NE: None
 Info: None
 Chunk: None

Surface: 返す
 Normalized: 返す
 Feature: 動詞,自立,*,*,五段・サ行,基本形,返す,カエス,カエス
 NE: None
 Info: None
 Chunk: <CaboCha.Chunk; proxy of <Swig Object of type 'CaboCha::Chunk *' at 0x0274A170> >


```  
