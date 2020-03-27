このドキュメントでは、すでにサポートが終わっているはずのXP+IE6の動作環境が必要になったらどうするかを説明する。  
  
最善の方法は実機を用意することだ。  
  
しかし、それが難しい場合がある。  
この記事ではVirtualPCを使用して仮想環境を用意する方法について説明する。  
  
なお、仮想環境では、シビアなタイミングで発生するバグの場合は再現が難しい可能性があるので注意すること。  
  
# VirtualPCによるレガシーな環境の作成  
今回はWindows7 HomeEditionをホストとしてXP+IE6.0の仮想マシンの環境を作成する。  
  
１．下記のページよりWindows Virtual PCを取得する  
http://www.microsoft.com/ja-JP/download/details.aspx?id=3702  
  
２．Microsoftのページより任意の仮想マシンをダウンロードする。  
http://www.modern.ie/ja/virtualization-tools#downloads  
(20181118更新) https://developer.microsoft.com/en-us/microsoft-edge/tools/vms/  
  
「Select a platform from the list below:」で「VirsutlPC for Windows 7」を選択後、Windows　IE6をダウンロードする。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/f839345b-3bc5-883f-bef8-0b961ebc22fd.png  
  
３． ダウンロードしたファイルをすべて同じフォルダに配置後、Exeを実行すると、ファイルが解凍されて、以下のファイルが作成されるので実行する。  
  
IE6 - WinXP.vmc  
  
４．デフォルトのユーザとしてXPUserが登録されているが、パスワードが未設定である。そのため、XPにログインするには「ツール→統合環境を無効にする」を選択しなければならない。  
  
統合環境を無効にした場合、ホストのクリップボード内容を仮想マシンにコピーしたりできなくなる。  
なので、仮想マシンにログインしたら、ただちにIEUserにパスワードを作成して統合環境を有効にしなければならない。  
  
なお、Vista 以降の仮想マシンの場合のIEUserのパスワードは次に記述してある。  
https://modernievirt.blob.core.windows.net/vhd/virtualmachine_instructions_2013-07-22.pdf  
  
５．HTTP経由でホストに接続するにはネットワークの設定が必要である。  
ホスト側のWin7にMicrosoft Loopback Adapterを導入する。  
http://blogs.wankuma.com/hatsune/archive/2010/05/13/189032.aspx  
  
次にVirtualPCのネットワークの設定をする。  
https://qiita-image-store.s3.amazonaws.com/0/47856/2faffda0-0e89-f576-66da-3e618a40e3af.png  
  
  
この時点で、Win7と仮想PCのXpでipconfigを行うとそれぞれに新しいネットワークアダプタとIPが割り当てられていることが確認できる。 そのIPを使って互いに通信できる。  
ただし、ファイアウォールを適切に設定しないとVista以降はPINGが通らないので注意すること。  
  
# 削除の方法  
下記のファイルを削除すればよい。  
C:\Users\XXXX\Virtual Machines\IE6 - WinXP.vmcx  
IE6 - WinXP.vmc  
IE6 - WinXP.vhd  
  
再度、使いたい場合は圧縮ファイルを解凍して同じ手順で再作成できる。  
この時、使用期限を含めたすべての情報は初期化される。  
  
  
  
# 制限事項  
この方法では次のような制限がある。  
  
・１月でアクティベートしないと使えなくなる。  
  
・英語のみしか使用できない。  
このことは、日本語のOffice等がインストールできない場合がある。  
(Office2000の日本語版はインストールできたがOffice2010の日本語版はインストールできず、英語版はインストールできた)  
  
なお、Fontを選択することが可能なアプリケーションは次の手順で日本語化できる。  
  
(1)日本語フォントを取得する  
http://ipafont.ipa.go.jp/ipafont/download.html  
  
今回はipag.ttfを使用した  
  
(2)フォントをインストールする。(1)で取得したフォントを次のディレクトリにコピーする。  
C:\WINDOWS\Fonts  
  
(3)ウィンドウズの[Display Properties]の[Advanced Appearance]でFontを指定する。  
https://qiita-image-store.s3.amazonaws.com/0/47856/c01a248d-111c-254e-2869-9daead82869a.png  
  
フォントを指定する箇所は以下の通りである。  
・タイトルバー  
・メニュー  
・メッセージボックス  
  
あとは、各アプリケーションの設定でフォントを指定すればよい。  
https://qiita-image-store.s3.amazonaws.com/0/47856/663b78bc-fe48-4e37-cdad-df63c694542b.png  
  
・キーボードが英語配列なので使いづらいし、当然、日本語入力はできない。  
なので、文字入力はホスト側で行ってクリップボード経由で転送した方が楽である。  
  
以下の方法で、日本語入力が行えるらしいが未検証。  
http://doratech.jugem.jp/?eid=40  
