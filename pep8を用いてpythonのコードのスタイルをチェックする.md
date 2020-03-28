# 2019.08.01  
古いので以下を参照してください。  
**カウボーイには嫌がられるPythonの話**  
https://qiita.com/mima_ita/items/cabcf014aa08e27c8de7  
  
# 概要  
本ドキュメントでは、Python コードのためのコーディングスタイルであるPEP8に違反していないかチェックする方法について説明する。  
  
PEP 8 -- Style Guide for Python Code   
http://legacy.python.org/dev/peps/pep-0008/  
  
訳:  
https://dl.dropboxusercontent.com/u/555254/pep-0008.ja.html  
  
  
# インストール方法  
  
PEP8  - Python style guide checker をインストールする  
https://pypi.python.org/pypi/pep8/  
  
```
easy_install pep8
```  
  
または  
  
```
pip install pep8
```  
  
これによりコマンドラインからpep8を実行できる。  
  
```
pep8 test.py
pep8 /test/directory
```  
ディレクトリを指定した場合はサブディレクトリもチェックする  
  
  
# 主なオプション  
  
|オプション名|説明|  
|:-----------|:---|  
|--version|バージョンを表示する|  
|-h,--help|ヘルプを表示する|  
|-v,--verbose|チェック中のファイル名などの状態メッセージを表示する。--vvでデバッグメッセージが表示される|  
|-q,--quiet|ファイル名のみ表示する。-qqはなにも表示しない。|  
|--first|同じエラーの場合、最初のみ表示する|  
|--exclude=patterns|除外するファイル名やディレクトリ名のパターンを記述する。コンマで区切ることで複数していできる。<BR>デフォルト：.svn,CVS,.bzr,.hg,.git,__pycache__|  
|--filename=patterns|ディレクトリを検索する場合に、ここで指定したパターンのファイルのみ検索する。コンマ区切りで複数指定できる。<BR>デフォルト: *.py|  
|--select=errors|エラーや警告を指定する <BR>例：E,W6|  
|--ignore=errors|指定のエラーを無視する <BR>例：E,W6|  
|--show-source|各エラーにソースを表示する|  
|--show-pep8|各エラーにPEP8の説明を追加する。--firstと伴につかったほうがいい|  
|--statistics|エラーや警告の数を集計して最後に表示する|  
|--count|最後にエラーと警告の総数を表示する|  
|--config=path|設定ファイルの場所を指定できる。|  
  
  
# 設定ファイル  
configオプションで指定できる設定ファイルには、各オプションの値を指定することができる。  
  
```
[pep8]
ignore = E111
```  
  
  
# Pythonからの使用  
pep8をimportすることでPythonから使用することもできる。  
  
```py
import pep8
pep8style = pep8.StyleGuide(quiet=True)
ret = pep8style.check_files(['test.py']);
print ret.total_errors
```  
  
# Jenkinsからの利用  
Jenkinsのプラグインである、Violationsを使用すれば集計が可能である。  
  
![pep8_2.png](/image/df3ba6ef-e494-f96c-54fe-bf4e3084aa91.png)  
  
  
シェルスクリプトから実行する場合は次のように先頭に「#!/bin/sh」を記述するのと最後にエラーコードを返さない処理を行う必要がある。  
  
```
# !/bin/sh
pep8 /share/py/test.py > ${WORKSPACE}/test.txt
echo "....finished"
```  
  
このような処理をする必要がある理由としては、下記のページを参照の事。  
[Jenkinsのシェルの実行について](http://qiita.com/mechamogera/items/f689b95670127d5bf046 "Jenkinsのシェルの実行について")  
Violationsの設定は下記の通り。  
  
![pep8_1.png](/image/4e6d8492-359f-873c-5cdd-640fa296d1b5.png)  
  
XMLで出力する必要はない。  
  
  
  
