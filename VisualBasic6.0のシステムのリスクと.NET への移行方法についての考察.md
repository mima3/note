ここでは現在も稼働しているVisualBasic6.0のシステムのリスクと、どのように.NETへ移行したらよいかを考察します。  
  
このドキュメントをPowerPointで表現したものは下記にあります。  
http://www.nicovideo.jp/watch/sm22232272  
http://needtec.sakura.ne.jp/doc/VisualBasic6system.pdf  
  
# VisualBasic6.0を使用し続ける場合のリスク  
VisualBasic6.0を使用したリスクには大きくわけて以下のリスクが存在します。  
  
・サポートに関するリスク  
・開発要員に関するリスク  
  
## サポートに関するリスク  
VB6ランタイムはWindows8まで保障されていますが、Grid32.dll,graph32.ocxなどの一部のランタイムは非サポートになりました。  
  
IDE(統合開発環境)はWindows7以降で非サポートです。実験結果として動作するという報告はありますが、あくまで非サポートです。  
  
WindowsVista,WindowsServer2008,Windows7,Windows8に関するVisualBasicのサポート状況は下記を参考にしてください。  
http://msdn.microsoft.com/ja-jp/vstudio/ms788708.aspx  
  
開発環境が非サポートになったということは、サードパーティが提供するOCX,DLLのサポート状況が不透明になります。  
マイクロソフトがVB6の開発環境を非サポートにしている状況で、サードパーティが、OCXやDLLのサポートを続けるとは考えづらいです。  
  
また、64ビットプロセスのサポートも行いません。  
64ビットマシンであっても32ビットプロセスとして動作することを義務付けられます。このことはプロセスのメモリの上限が最大4Gになることを意味しており、どんなにメモリを積んだとしてもそれ以上のメモリを１プロセスで使用することができなくなります。  
  
## 開発要員のリスク  
今後はVisualBasic6.0の開発要員を確保しづらくなることが予想されます。  
以下の図は2012/4/27にIPA独立法人情報処理推進機構が発表した、ソフトウェア産業の実態把握に関する調査の結果です。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/df8c40b5-323b-73f6-ef24-5d39c29671f3.png  
  
これは新規で人手で作成したプログラミング言語の比率をあらわします。  
VB6はCOBOLの0.4％以下のその他に分類されています。  
  
このことより、新規案件でVisualBasic6.0が採用されていないことを表します。  
このことにより、新規に業界に入った人間は、VisualBasic6.0を扱うことが少なくなることになります。つまり、VisualBasic6.0を扱える人材の年齢が上がっていくことになり、これは単価の上昇を意味します。  
このことより、将来、安価な労働力の確保ができなくなることが想像できます。  
  
# 対応策  
VisualBasic6.0でシステムを開発し続けることにはいくつかのリスクがあることを説明しました。  
ここでは、今後どのようにすべきかを考察します。  
  
考えるられる方法として３つの方法が存在します。  
  
・リスクを把握した上で使い続ける  
・.NETに移行する  
・新規機能のみ.NETに置き換える  
  
## リスクを把握した上で使い続ける  
OSのバージョンアップやWindows Updateを行わない前提であれば可能な選択肢です。  
  
たとえば、工場の機械を制御するシステムのように、インターネットにつながず、他のOfficeなどのアプリケーションを使用しない閉じた環境であれば、OSのバージョンアップやWindowsUpdateなどのセキュリティを考慮しないですむので、VB6を使い続けることができるかもしれません。  
  
ただし、更新しないつもりでも、マシンが壊れて新しいマシンを入れる場合に古いOSを入手できないというリスクや、和暦をつかっている場合は元号が変わった場合にOSに対してなんらかのパッチを当てなければならないリスクが存在します。  
  
## .NETに移行する  
これはMicrosoftが推奨している方法で技術的には正しい選択肢でしょう。  
しかし、コストの問題が発生します。  
VB6のソースコードから.NETのソースコードに変換する、コンバートツールは存在しますが、その精度は低く、最終的には人間が確認する必要があります。  
仮にうまくいっても全テストを実施しなければなりません。  
  
潤沢な予算がなければ選択するのは難しいでしょう。  
  
## 新規機能のみ.NETにおきかえる  
上記2案の折衷案です。  
現状のコードをなるべくいじらずにいかしつつ、問題が発生した箇所や、新規機能を.NETに置き換えていきます。  
これにより、段階的な.NETへの移行作業を行えます。  
  
基本的にはCOMで.NETの機能をラップしてVBから使用します。  
.NETでCOMを作成する方法については以下と同様の手順で作成することができます。  
  
 __VBAまたはVBSからCOM経由で使用できる.NETのライブラリの作成方法__   
http://qiita.com/mima_ita/items/efcd1a6ea86f09047984  
  
もし、フォームやコントロールを作成する必要がある場合、Microsoftより提供されているInterop　Forms Toolkit を利用すれば、.NETで作成したコントロールやフォームをCOM経由でVBから使用することが簡単になるでしょう。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/9131b241-0b26-797e-6f02-d7c5963fafb9.png  
  
Interop　Forms Toolkit 2.0 Tutorial  
http://www.codeproject.com/Articles/18954/Interop-Forms-Toolkit-2-0-Tutorial  
  
http://www.microsoft.com/en-us/download/details.aspx?id=3264  
  
  
### Windows7 + VB2008.NETでのInterop Forms Toolkitの使用方法  
#### COMの作成方法  
以下でInterop Forms Toolkitを使用してVB2008.NETのCOMを作成してみます。  
  
１．Interop Forms Toolkitをインストールして、管理者権限としてVisualStudioを起動します。これはCOMの登録に管理者権限が必要だからです。  
  
２．新規プロジェクトで[VB6 Interop UserControl]のテンプレートを選択します。  
https://qiita-image-store.s3.amazonaws.com/0/47856/542e9bd6-69e4-1188-1287-a5ca62234b99.png  
  
３．テンプレートを選択してプロジェクトを作成すると、VBB.NETでCOMのコントロールやインターフェイスを実装できるようになります。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/2b87c1ed-9d09-2a16-9e38-963433356b41.png  
  
４．VB6から使用したい機能をCOM参照可能なクラスのCOM参照可能なパブリックプロシージャとして実装してください。  
https://qiita-image-store.s3.amazonaws.com/0/47856/9b931afb-60d1-d447-ad05-aa9f26109995.png  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/7b40aa16-65b0-284e-2fea-099cc01d8b49.png  
  
５．実装が終わり、ビルドを行う時に、COMとして登録されるように、プロジェクトの設定で「COM相互運用機能の登録」を選択してください。  
https://qiita-image-store.s3.amazonaws.com/0/47856/3aa05e98-8930-2736-e9db-8ec4cf7670ca.png  
  
もし、VisualStudioのない環境でCOMを登録するにはRegasmコマンドを使用します。  
  
```
Regasm AssemblyName.dll /tlb: FileName.tlb /codebase
```  
  
http://support.microsoft.com/kb/817248/ja  
  
#### COMの使用方法  
１．VB6やVBAでCOMを使用するには「参照設定」で作成したDLLを選択します。  
コントロールとして使いたい場合は、ツールボックスから、その他のコントロールで追加できます。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/6c8d3496-ade2-da98-d09e-6beb5d7edf06.png  
  
  
２．VB6やVBA側でオブジェクトを作成して使用することが可能になります。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/65572227-716e-490a-c5df-55b54fdf9674.png  
  
  
#### デバッグの方法  
デバッグの方法としてはCOMを実装したソリューションにテストようのプロジェクトを追加してそこで単体テストをすると楽です。  
同じソリューションであれば、ブレイクやウォッチが使用できてデバッグが楽です。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/08ca7f6b-0539-2bec-b38e-03753fcd692d.png  
  
  
あるいは、VisualStudioでVB6.0で作成したプロセスにアタッチすることで、.NETで作成したCOMのデバッグが行えます。  
VB6.0側と.NET側のどちらにバグの原因があるかわからない場合はこの方法がよいでしょう。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/d482415d-88bc-f012-c364-dac715034227.png  
  
#### 注意点  
Interop Forms Toolkit2.1を動かした際に気になった点を以下に示します。  
  
・Interop Forms Toolkit 2.1インストール時にHelp Fileのショートカットがプログラムファイルに作成できない場合がある。（インストールは正常にできる）  
  
### VB6で32bitを超えるメモリ空間にアクセスしたい場合  
１プロセスではむりです。  
.NETで64ビットのアウトプロセスのCOMサーバを作成して、別プロセスとして独立させ、それをVB6から操作してください。  
 __.NETにおける64ビットプロセスと32ビットプロセスについて__   
http://qiita.com/mima_ita/items/57d7c1101543e214b1d6  
  
### まとめ  
このようにCOMを利用することで一部の機能の.NET化を行うことができるので、移行の作業量を抑えながら,段階的な.NETへの移行が可能になります。  
  
なお、COMから.NETを使用する場合、.NET フレームワークをCOMを使用する際に読み込むのでVB6.0単体の時より起動時間が掛かる可能性があるので注意してください。  
  
  
# 参考  
 __Windows Vista、Windows Server 2008、Windows 7、および Windows 8 に対するVisual Basic 6.0 のサポートに関する声明__   
http://msdn.microsoft.com/ja-jp/vstudio/ms788708.aspx  
  
 __Visual Basic 6.0 から Visual Basic .NET または Visual Basic 2005 アセンブリを呼び出す方法__   
http://support.microsoft.com/kb/817248/ja  
  
 __COM 相互運用 (Visual Basic)__   
http://msdn.microsoft.com/ja-jp/library/6bw51z5z.aspx  
  
 __Yuya Yamaki’s blog Microsoft InteropForms Toolkit 1.0__   
http://d.hatena.ne.jp/Yamaki/20061101  
  
  
 __Interop Forms Toolkit 2.0 Tutorial__   
http://www.codeproject.com/Articles/18954/Interop-Forms-Toolkit-2-0-Tutorial  
