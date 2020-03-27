最近始めたAndroidについてのメモです。  
  
このメモのおかげで、貧弱な坊やだった私でも一月でアプリがくめたよ。やったねたえちゃん。  
  
 **Bluetoothで監視カメラ**   
https://play.google.com/store/apps/details?id=jp.ne.needtec.bluetoothcameraclient  
  
# 開発環境  
## AndroidStudio  
最近はEclipseでやるより、AndroidStudioで開発するようだ。  
https://developer.android.com/sdk/index.html  
  
以下の環境で、モッサリとではあるが、動作する。  
CPU:Intel(R) Core(TM) i5CPU M450 @2.4GHz  
メモリ:8GB  
OS:Windows7 64bit  
  
## アプリケーションの実行  
アプリケーションを実行する方法は３つある。  
1.実機を使用する方法  
2.エミュレータ―を利用する方法  
3.VMWarePlayerにAndroidOSを入れる方法  
  
### 実機を使用する方法  
一番楽。  
USBケーブルでつなげて、AndroidStudioからRunを実行すれば、実行対象のデバイスを選択できる。  
https://qiita-image-store.s3.amazonaws.com/0/47856/9d717152-57b7-7ff9-4cb8-627cd0fa7087.png  
  
### エミュレータを利用する方法  
AndroidStudioからエミュレータを起動することができる。  
実機がなくても動くのは大きなメリットではあるが、速度が遅すぎて話にならない。  
  
また、Windowsの場合、エミュレータの割り当てメモリ量によって落ちたりする。  
http://stackoverflow.com/questions/7222906/failed-to-allocate-memory-8  
メモリ量は、小さすぎても、大きすぎても使えない。  
  
貧弱なPCにはおすすめしない。  
  
### VMWarePlayerにAndroidOSを入れる方法  
VMWarePlayerにAndroidOSを入れて実行する。  
エミュレータ―よりずっとはやい!!  
ただし、センサーやBluetoothのサポートは行われていない。  
  
この手順については下記を参照  
http://www.japan-secure.com/entry/blog-entry-434.html  
http://blog.fujiu.jp/2011/05/android-vmware-playerandroid.html  
  
AndroidStudioでVMWare上のAndroidOSを認識させるには以下のような手順が必要  
  
1.AndroidOS側でALT+F1を押してコマンドプロンプトを開き、下記のコマンドでipを調べる  
  
```
ifconfig eth0
```  
  
2.ホスト側で以下のコマンドを実行して、VMWare上のAndroidと接続させる。  
  
```
C:\Users\ユーザー名\AppData\Local\Android\sdk\platform-tools\adb connect 192.168.67.146:5555
```  
  
3.Android StudioでRunを実行するとデバイスとして表示される。  
https://qiita-image-store.s3.amazonaws.com/0/47856/3cd4c4c2-0406-e6d6-eee9-574207d416d2.png  
  
なお、VMWarePlayer上で使用できる特殊なショートカットキーは以下の通り  
  
|キー操作|動作|  
|:--|:--|  
|CTRL+ALT|ホスト側のコンピュータに操作を戻す|  
|ALT+F1|コマンドプロンプト|  
|ALT+F7|コマンドプロンプト終了|  
|F9を二回連打|上を上にした画面になる|  
|F10を二回連打|上を下にした画面になる|  
|F11を二回連打|上を左にした画面になる|  
|F12を二回連打|上を右にした画面になる|  
  
## 仮想環境でのセンサーの利用  
実機であれば実機についているセンサーをそのまま使える。仮想環境の場合はどうするべきか？  
  
Sensor Simulatorを利用すれば、一応不可能ではない。  
https://code.google.com/p/openintents/wiki/SensorSimulator  
  
ソースコードも公開されているので自分で調整も可能。  
https://github.com/openintents/sensorsimulator  
  
使い方としては下記のページを参照  
http://blog.goo.ne.jp/jsp_job/e/8d524f55e621899b7a7fcf85a14fa8d5  
  
対象の仮想環境にSensorSimulatorをインストールして、ホストOSのGUIから、その設定値を伝える。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/d5069492-fc48-bf6e-0fe4-849fa3a967ea.png  
  
センサーの数値を明示できる分、便利かもしれないが、これは、アプリケーション側のコードでクラス名を修正する必要がある。  
  
```
android.hardware.Sensor
　　→org.openintents.sensorsimulator.hardware.Sensor
```  
  
多分実際に使うとしたら、この当たりをクラスでラップして、実機と仮想環境で切り分けるようにしとかんとダメだろう。  
  
## Jarの利用  
他人が作ったJarファイルはlibsフォルダに入れておけばAndroidStudioが認識するようになる。  
  
このあたり参照。  
http://rakuishi.com/archives/5768/  
  
## NDKの利用  
NDKのサポートは将来機能になっているが、現時点でも、NDKで作成したライブラリも利用できる。  
  
NDKでビルドしたライブラリをAndroid Studio1.1で使用するには以下を参照。  
http://qiita.com/mima_ita/items/6cee54fefcdb96999ab6  
  
おそらく、バージョンアップに伴って使い方が変わるだろう。  
(実際、1.0の記事がすでに役にたたなくなっている）  
  
## デバイスとファイルのやり取りを行う方法  
AndroidDeviceMonitorを使用すればファイルのやり取りが行える。  
ただし、権限の関係で全部のディレクトリの内容が見えるわけでもなさそう。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/233b5bdb-f49f-b008-6073-7d134bc44212.png  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/226ffc38-51d8-a8db-cfea-c2541b300040.png  
  
## デバイス上のスクリーンキャプチャーと動画の作成  
「Screen Capture」ボタンを押せば、現在接続中のデバイスのスクリーンショットを任意の名前で保存できる。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/1499ca52-b594-d38e-af5d-f17ab0344997.png  
  
その下の「Screen Record」ボタンでは、動画として保存可能。  
  
## メモリの使用状況  
「Tool->Android->Memory Monitor」を実行することで、その指定のプロセスのメモリ使用状況が確認できる。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/32c06fc4-9a2c-6601-df07-d11bdcd93590.png  
  
  
詳細な使用状況を調査するには以下のようにする。  
1.AndroidStudioのDevicesより、調査対象のプロセスを選択して、「Dump Java Heap」を選択する。  
https://qiita-image-store.s3.amazonaws.com/0/47856/aa6fac2a-f89c-efc6-7502-4d01b4acc4ee.png  
  
2.MATでそのダンプを解析する。  
https://qiita-image-store.s3.amazonaws.com/0/47856/aa44837d-f9d5-6684-5d82-f13457b1e132.png  
  
MATは下記からダウンロードできる。  
http://www.eclipse.org/mat/downloads.php  
  
## ビルド  
AndroidStudioのビルドはGradleっていう言語を利用して記述されている。  
XMLとかとちがってプログラミング言語なので、わかりやすいかもしれない。  
  
AndroidのGradleのリファレンスは下記のページの「Plugin Language Reference」からダウンロードできる。  
http://developer.android.com/tools/building/plugin-for-gradle.html  
  
## 便利なショートカットキー  
「CTRL+ALT+L」で書式を自動で整形してくれる  
https://qiita-image-store.s3.amazonaws.com/0/47856/a2d5cef0-faf9-fa6a-4aa9-dcae8e0f753b.png  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/e8e50dfb-0bd1-ddbd-3eba-af8246275077.png  
  
その他、次のようなショートカットキーがある。  
http://qiita.com/sugoi_wada/items/db449d5cbb5c83cb586c  
  
## 静的解析  
AndroidStudioは常時、静的解析を行い、まずい書き方をしているコードを教えてくれる。  
エディタの右側に黄色くなっている箇所をマウスオーバーすると、なにがまずいか教えてくれる。  
https://qiita-image-store.s3.amazonaws.com/0/47856/baf7da36-e31f-0ace-f3a2-21ffc72a2738.png  
  
  
## テスト  
Android　Studio1.1からUnitテストをデフォルトでサポートしている。  
http://tools.android.com/tech-docs/unit-testing-support  
  
基本的にJUnitっぽい。そして、GUIのテストもできるっぽい。  
http://developer.android.com/tools/testing/activity_testing.html  
  
# 開発TIPS  
## アプリケーションのアイコンの変更  
下記のアイコンが必要になる。  
・MDPI 48x48  
・HDPI 72x72  
・XHDPI 96x96  
・XXHDPI 144x144  
  
あとは、下記参照。  
http://androidstudio.hatenablog.com/entry/2014/07/23/111303  
  
## GoogleMap  
GoogleMapを使用したアプリも作れるが、GoogleでのAPIの登録が必要。  
http://maruta.be/intfloat_staff/168  
  
現在位置を取得するには、FusedLocationApiを使って現在地を取る。  
このあたりを参照。  
http://blog.teamtreehouse.com/beginners-guide-location-android  
http://javapapers.com/android/android-location-fused-provider/  
  
VMWarePlayerではGoogleMapが上手く表示できない。  
エミュレータはOK.  
  
## デザインについて  
Google様が下々にデザインガイドラインをお示しくださっている  
http://developer.android.com/design/index.html  
  
## アプリケーションの権限  
アプリケーションインストール時に固有ユーザーを作成している。  
おそらく、これで、別のアプリからの干渉をさける。  
マニフェストファイルのsharedUserIdを使うとインストール時のユーザーを同じにできる。  
  
```
android:sharedUserId="http://jp.co.needtec.bmical"
```  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/e0bf23a6-7ab4-1aae-8fe6-23fb5492a6f9.png  
  
## AlermManager  
AlermManagerの～WALK_UPはスリープ中でもイベント発火するっぽい。  
でも,onReceiveぬけるとすぐ二度寝するので、WalkLockが必要っぽい。  
  
## アプリ間のデータ共有  
コンテントプロバイダって機能を使ってあるアプリが持っている機能を別のアプリに公開する。  
SharedPreferences のMODE_WORLD_～は非推奨になっている。  
  
## クリック関係のイベントの発火順序  
OnTouch（ACTION_DOWN)  
OnLongClick(長おししたとき）  
OnTouch(ACTION_UP)  
OnClick  
  
長押しの時間ってのはてのはViewConfigurationのgetLongPressTimeoutっぽい。デフォで500m秒。  
http://qiita.com/y-takano/items/58cd0060089bd4775780  
  
## Widgetの重み  
android:lay_outweightをつけるとウィジェットに重みをつけられる。  
これを利用すれば、どっちを優先して大きくするか指定できる。  
  
## Bluetoothの操作  
以下の記事参照  
https://sites.google.com/a/gclue.jp/android-docs-2009/bluetoothapurino-kaihatsu  
  
Windowsと通信するときはエンディアンに注意。  
稼動CPUを問わずJava仮想マシンについてはビッグエンディアンなので、Windows側では「_byteswap_ulong」などでバイト列を調整する。  
  
Java同士なら意識する必要なし。  
  
また、通信速度はそんなに速くない。小さい画像を転送する分には気にならないが、大きなサイズだと遅すぎる。接続範囲も、狭い。  
（窓の外にAndroid端末おいて操作する程度はできるが、Android端末もって部屋から出ていくと切断される）  
  
## カメラの制御  
カメラの制御については下記参照。  
http://qiita.com/zaburo/items/d9d07eb4d87d21308124  
  
ただし、2.1-Update 1が対象の場合、プレビュー周りで落ちる場合があるので、以下を参考にする。  
https://code.google.com/p/android/issues/detail?id=7909  
  
どうも、surfaceChangedでのsetPreviewSizeが具合悪いようだ。  
onResumeで使用する分には問題なし。  
  
https://github.com/mima3/BluetoothCamera/blob/e547e0980d7107b7ae11b2fee316d3e4897a9cfe/BluetoothCameraClient/app/src/main/java/jp/ne/needtec/bluetoothcameraclient/CameraPreviewActivity.java  
  
また、カメラのプレビューの画像はyuv420spになっている。  
これをビットマップに変換する場合は、下記を参考。  
http://qiita.com/GeneralD/items/68142abb852c392db236  
  
もし、WindowsのBITMAPにする場合は上下を反転させる必要がある。  
https://github.com/mima3/BluetoothCamera/blob/e547e0980d7107b7ae11b2fee316d3e4897a9cfe/BluetoothCameraServerWin/BluetoothCameraServer/BluetoothCameraReceiver.cpp  
  
## ORMの使用  
SQLiteのORMとして、ORMLiteがある。  
http://ormlite.com/releases/  
http://qiita.com/radiocatz/items/5f1dce3f8c5faa55e6f6  
  
jarをlibsにつっこめば使える。  
  
## NFCタグに情報を読み書きする方法  
書き込み可能なタグはネット通販などで20枚1200円程度で購入可能なので、社員証の代わりとかで使えそう。  
  
http://www.atmarkit.co.jp/ait/articles/1211/27/news072.html  
  
C++で実装しようとすると銭が必要そう。  
http://www.sony.co.jp/Products/felica/business/products/ICS-D004_002_003.html  
  
  
# リリース方法  
手順は以下の通り  
1.Android StudioでAPKファイルを作成する。  
http://androidstudio.hatenablog.com/entry/2014/07/26/154043  
  
2.GooglePlayに登録する。  
https://play.google.com/apps/publish/  
  
・開発者の登録料として25$をクレジットカードで支払う必要がある。  
  
・電話番号の登録には国コードが必要。日本は「+81」をいれとく。  
  
・アプリを公開したい場合、「なぜ公開できないか」という情報が表示されているので、それにあわせて対応していけば、公開できるようになる。  
  
・アプリのバージョンアップ時にはbuild.gradle中のversionCodeを前回リリース時より大きくする必要がある。  
  
・「アルファ版 / ベータ版テストと段階的公開の使用」として特定のユーザーに絞って公開することもできるっぽい。  
https://support.google.com/googleplay/android-developer/answer/3131213?hl=ja  
  
この際、米国の輸出法に従う必要がある。  
暗号を利用している場合は注意。  
たとえば、SSLでの通信も対象になる。  
例外事項として、パスワードの暗号化、認証、デジタル署名を目的に使用するのは対象外。  
  
  
 **Google Marketにある輸出規制って何？**   
http://synchack.com/memo/index.php?Tips%2FGoogle%20Market%A4%CB%A4%A2%A4%EB%CD%A2%BD%D0%B5%AC%C0%A9%A4%C3%A4%C6%B2%BF%A1%A9  
http://www.cistec.or.jp/service/iinkaidayori/houkokusho2010/data_kamotsu/kamotsu5-3-1.pdf  
  
 **暗号化に関する輸出制限について**   
https://msdn.microsoft.com/ja-jp/library/windows/apps/xaml/hh694069.aspx  
  
  
  
# 気付いた点  
・変化がくっそ早い分野なので、古い本とか使用すると、間違い探しになります。なるべく新しい本を買った上で、あんまり信用しないで使用しましょう。  
　→ゆえに初心者が初手Android開発やると嵌ると思います。  
  
・いきなりBluetoothとかのハードウェアを使うプログラムはやめよう。  
　→仮想環境で動かないので、実機を用意する必要がでます。  
　 金かけれるならともかく、おためしでやるにはつらいかと。  
  
  
・いきなりNDKを使ったプログラムはやめよう。  
　→むちゃくちゃきつい  
  
・公開時の審査は数時間でおわる。アップルと違って緩いってはっきしわかる。  
  
・GUIのプログラムを学習するときは、最初、XMLを無視して、コード上でGUIを設置した方が直観的にわかりやすい。  
　ある程度、つかんだ後に、XMLでも定義するという流れがいいかと。  
