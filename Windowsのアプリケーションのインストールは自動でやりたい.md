# 目的  
よくプロジェクトの新規メンバーが入った時、各種アプリを手でポチポチインストールすることがあります。  
そして、どんなに丁寧に手順書を書いても、設定や順番をミスる人はミスって、何回か重大な局面で環境を色々見直すはめになることが稀によくあります。  
  
今回は、Windowsのインストールを自動化できるかどうか検討してみます。  
  
なお、この記事でコマンドをいくつか実行していますが、すべて管理者権限のあるコマンドプロンプトより実行しています。  
  
# インストーラについて  
Windowsのインストーラは大きくわけて２種類あります。  
Setup.exeのように拡張子がexeのものと、拡張子がmsiのものです。  
  
msiはユーザー応答の有無などの基本的なオプションについて共通的になっています。  
exeの場合、それぞれが異なる可能性もありますし、また、ユーザー操作なしでインストールが行えない場合もあります。  
  
## 拡張子がexeのインストーラの操作例  
まず、GUIなしでインストールできるオプションがあるかどうかを調べるためにセットアップのヘルプを見ます。  
たとえば、WinMergeの場合は以下のようになります。  
  
```
WinMerge-2.1.6.4-Setup.exe /?
```  
  
![image.png](/image/a64ab5a4-95aa-a048-b379-7bf0314f10a0.png)  
  
これを読めば、/SILENT、または/VERYSILENTを付与して実行すればGUIの操作なしでインストールできると考えられます。  
では実行してみましょう。  
  
```txt
WinMerge-2.1.6.4-Setup.exe /VERYSILENT
```  
  
GUIなしでWinMergeがインストールできることが確認できます。  
![image.png](/image/b543fc4b-b758-52fe-bb56-85ef9ec9bd0c.png)  
  
## 拡張子がmsiのインストーラの操作例  
msiexecでMSIファイルの操作が行えます。このコマンドについては/?でヘルプを確認するか、公式のヘルプを参考にしてください。  
  
  
```
Windows R インストーラー. V 5.0.7601.23593 

msiexec /Option <必須パラメーター> [省略可能なパラメーター]

インストール オプション
	</package | /i> <Product.msi>
		製品をインストールまたは構成します。
	/a <Product.msi>
		管理用ツール - ネットワーク上の製品をインストールします。
	/j<u|m> <Product.msi> [/t <変換一覧>] [/g <言語 ID>]
		製品をアドバタイズします - すべてのユーザーには m、現在の
		ユーザーには u を指定します。
	</uninstall | /x> <Product.msi | 製品コード>
		製品をアンインストールします。
表示オプション
	/quiet
		Quiet モード - ユーザーの操作なし
	/passive
		無人モード - 進行状況バーのみ
	/q[n|b|r|f]
		ユーザー インターフェイスのレベルを設定します。
		n - なし
		b - 基本
		r - 簡易
		f - 完全 (既定)
	/help
		ヘルプ情報
再起動オプション
	/norestart
		インストール完了後に再起動しません。
	/promptrestart
		再起動が必要な場合は、ユーザーに再起動を要求します。
	/forcerestart
		常に、インストール後コンピューターを再起動します。
ログ オプション
	/l[i|w|e|a|r|u|c|m|o|p|v|x|+|!|*] <LogFile>
		i - 状態メッセージ
		w - 致命的ではない警告
		e - すべてのエラー メッセージ
		a - 操作のスタートアップ 
		r - 特定の操作の記録
		u - ユーザーの要求
		c - UI パラメーターの初期値
		m - メモリ不足または致命的な終了に関する情報
		o - ディスク領域不足メッセージ
		p - ターミナルのプロパティ
		v - 詳細出力
		x - 詳細デバッグ情報
		+ - 既存のログ ファイルに追加
		! - 各行をログにフラッシュ
		* - v オプションと x オプションを除くすべての情報をログに記録します。
	/log <ログ ファイル>
		/l* <ログ ファイル> と指定したときと同じ情報がログに記録されます。

更新オプション
	/update <Update1.msp>[;Update2.msp]
		更新を適用します。
	/uninstall <修正プログラム コード GUID>[;Update2.msp] 
	/package <Product.msi | 製品コード>
		製品の更新を削除します。
修復オプション
	/f[p|e|c|m|s|o|d|a|u|v] <Product.msi | 製品コード>
		製品を修復します。
		p - ファイルが見つからない場合のみ
		o - ファイルが見つからない、または古いバージョンが
		    インストールされている場合 (既定)
		e - ファイルが見つからない、同じバージョンまたは古い
		    バージョンがインストールされている場合
		d - ファイルが見つからない、または違うバージョンが
		    インストールされている場合
		c - ファイルが見つからない、またはチェックサムと計算
		    された値が一致しない場合
		a - すべてのファイルをインストールする
		u - すべてのユーザー固有の必須レジストリ エントリ (既定)
		m - すべてコンピューター固有の必須レジストリ エントリ (既定)
		s - すべての既存のショートカット (既定)
		v - ソースから実行して、パッケージをローカルに再キャッシュする
パブリック プロパティの設定
	[PROPERTY=プロパティ値]

コマンド ラインの構文の詳細については、Windows (R) インストーラー SDK を参照してください。

Copyright (C) Microsoft Corporation. All rights reserved.
Portions of this software are based in part on the work of the Independent JPEG Group.
```  
  
https://docs.microsoft.com/ja-jp/windows/win32/msi/standard-installer-command-line-options  
https://docs.microsoft.com/ja-jp/windows/win32/msi/command-line-options  
  
このことより以下のコマンドを使えばいいとわかります。  
  
/i でmsiファイルを指定する。  
/passive で「無人モード - 進行状況バーのみ」の表示になります。  
/qn でユーザー インターフェイスをなしにできます。  
/norestartで再起動をおこないません。  
  
つまり以下のような実行を行うことで自動でインストールが行えます。  
今回はTortoiseSVN-1.12.2.28653-x64-svn-1.12.2で実験します。  
  
```text
msiexec /i TortoiseSVN-1.12.2.28653-x64-svn-1.12.2.msi /passive /qn /norestart
```  
  
![image.png](/image/72267e3b-0eb4-b877-9379-7e7752c99e38.png)  
  
  
## インストール時のオプションを変更したい  
上記のインストールを行った場合、コマンドラインからSVNコマンドが実行できません。  
これは、デフォルトのインストールの動作が以下のように「command line client tools」が除外されているためです。  
  
![image.png](/image/9922019f-bcfa-dfa5-06aa-a94e65b7e2d0.png)  
  
  
このオプションを変更するには各アプリケーションが持っている特定のプロパティを指定する必要があります。  
これを調べる方法については以下にヒントがあります。  
  
https://serverfault.com/questions/384904/retrieving-public-properties-from-an-msi-file  
YenForYang氏の回答がベターです。  
  
氏はインストール時のログを以下のコマンドで取得し、それを参考にプロパティを探すべきと言っています。  
  
```
msiexec /lp! <msi_property_logfile> /i <msi_name>
```  
  
たとえばTortoiseSVNの場合は以下のように行います。  
  
(1)下記のコマンドを実行する。  
  
```
msiexec /lp! log_default.txt /i TortoiseSVN-1.12.2.28653-x64-svn-1.12.2.msi
```  
  
(2)インストーラーが起動するので、何も変更せずにインストールを行う。  
  
(3)アインインストールする。  
  
(4)下記のコマンドを実行する。  
  
```
msiexec /lp! log_cli.txt /i TortoiseSVN-1.12.2.28653-x64-svn-1.12.2.msi
```  
  
(5)Command Line Toolを設定した状態でインストールをする。  
  
(6)作成されたログファイルを比較する  
![image.png](/image/2fcc46d3-88d1-fdd9-2425-1120f8ddf975.png)  
  
比較をしているとCommand Line Toolを有効にした場合は、ADDLOCALにCLIが追加されていることがわかります。  
  
すなわち以下のようなコマンドでCommand Line Toolを有効にしたインストールが行えます。  
  
```
msiexec /i TortoiseSVN-1.12.2.28653-x64-svn-1.12.2.msi /passive /qn /norestart ADDLOCAL=F_OVL,DefaultFeature,MoreIcons,CLI,CrashReporter,UDiffAssoc,DictionaryENGB,DictionaryENUS

```  
  
あとは必要に応じて端末の再起動を行えばSVNのコマンドラインツールが使えるようになります。  
  
# まとめ  
インストールをGUIなしで自動化できるヒントについてまとめました。  
すべてのアプリケーションで有効ではないですが、特定の条件下においては利用できると思います。  
  
  
