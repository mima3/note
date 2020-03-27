本ドキュメントはMicrosoftレポートを用いた.NET Frameworkによる帳票の作成について記述します。  
  
PDF版は下記からダウンロードできます。  
http://needtec.sakura.ne.jp/doc/msreport.pdf  
  
WORDで80ページほどあるので、PDFで読んだ方が楽かもしれません。  
  
# MicroSoftレポートとは？  
MicrosoftレポートとはVisual Studioが帳票の作成をサポートするために提供しているコントロールです。レポートのデザイン機能と、アプリケーションでレポート処理および表示をするためのReportViewerコントロールが提供されています。  
次の図はMicrosoftレポートが、どのように帳票を作成するかを簡単に表したものです。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/e8ea4637-81df-c250-1794-57880aad3e05.png  
  
 __データソース：__   
アプリケーションが使用するデータのことで特にアプリケーションで操作する必要のあることが明確なデータです。  
  
 __レポート定義：__   
帳票のレイアウトや、どのデータを作成するかをXMLで記述したものですVisual Studioのレポートデザイナを使用して変更が可能です。  
  
 __ReportViewerコントロール：__   
データセットとレポート定義より帳票を作成します。  
  
## 作成できる帳票  
Microsoftレポートは形式の帳票を作成できます。以下に、いくつかの例を紹介します。  
  
### 一覧形式  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/d8594d9d-07ec-aa8c-5e59-ea810ee3f809.png  
  
Excelのように一覧としてデータを表示します。  
各フィールドでは書式の指定や数式の指定が行えます。必要であれば、開発者は自分の目的にあった数式を自作して利用することもできます。  
  
### 単票形式  
https://qiita-image-store.s3.amazonaws.com/0/47856/8a8755a8-95ad-13bd-19ac-fed1f317e936.png  
  
単票としてデータの詳細を自由なフォーマットで表現します。  
数字や、文字だけでなく、画像も使用できます。  
  
### グラフ  
https://qiita-image-store.s3.amazonaws.com/0/47856/39031155-4365-6072-1faf-dc08a4d14a9b.png  
  
データをグラフとして表示することができます。  
棒グラフだけでなく、折れ線グラフや円グラフなど、様々な表現方法が実現できます。  
  
# 動作環境  
Visual Studio 2005以降でProfessional以上であれば使用できます。  
ExpressにはReportViewerコントロールは提供されていません。  
なお、マイクロソフトが提供するチュートリアルを行う場合、SQL Serverが別途必要になります。  
  
このドキュメントで紹介するチュートリアルは以下の開発ツールがインストール済みであることを前提とします。  
・Visual Studio2013 Professional  
・SQL Server2012 Express  
・SQL Server Management Studio(SSMSと略される場合がある)  
  
# チュートリアル  
Microsoftは下記のページにサンプルとチュートリアルを用意しています。  
 __サンプルとチュートリアル__   
http://msdn.microsoft.com/ja-jp/library/ms251686%28v=vs.90%29.aspx  
  
VisualStudio2013とSQLSERVER2012では、このチュートリアル通りおこなっても、サンプルデータすら作成できません。  
この章ではチュートリアルの補足を行います。  
  
## サンプルで使用するデータベースの構築  
チュートリアルで使用するサンプルを動かすにはSQLSERVERでテスト用のデータベースを作成する必要があります。  
  
 __チュートリアル : AdventureWorks データベースのインストール__   
http://msdn.microsoft.com/ja-jp/library/aa992075%28v=vs.90%29.aspx  
  
ここではテストデータのはいったSQLSERVERのデータベースを開発者のPCに構築するための手順が記述してあります。  
ただし、ここで記述している通りにやっても、ファイルのダウンロードすらできません。以下の手順を参考にしてSQLSERVERを構築してください。  
  
１．まずテスト用のデータベースを下記のサイトから取得します。  
 __AdventureWorks Sample Databases for the SQL Server 2012 Database Training Kit__   
http://sql2012kitdb.codeplex.com/releases/view/86805  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/233b5e03-a026-6831-b137-2b49d119d208.png  
  
  
２．ダウンロードしたMDFファイルをSQLSERVERにアタッチできる場所にコピーします。  
SQLSERVERをデフォルトの設定でインストールした場合は以下のパスになるでしょう。  
C:\Program Files\Microsoft SQL Server\MSSQL11.MSSQLSERVER\MSSQL\DATA  
  
もし使用しているSQLSERVERがEXPRESSの場合は次のパスになります。  
C:\Program Files\Microsoft SQL Server\MSSQL11.SQLEXPRESS\MSSQL\DATA  
  
コピーを行うさい、ダウンロードしたMDFファイルの「ブロックの解除」が行われていることを確認してください。これはファイルを右クリックしてプロパティで確認できます。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/f5460edd-64be-ecb5-8487-3f636ad131e4.png  
  
３．SQLServer　Management Studioを起動してローカルのSQLSERVERに接続します。  
「localhost」のWindows認証で接続できるはずです。  
もし、Expressの場合は「Localhost\SQLEXPRESS」を指定してください。  
４．オブジェクトエクスプローラーで「データベース」ノードを右クリックして「アタッチ」を選択してください。  
５．「追加」をクリックしてコピーしたMDFファイルを選択してOKボタンを押します。  
６．「データベースの詳細」リストからファイルの種類で「ログ」を選択して削除します。  
７．OKボタンを押せばデータベースはアタッチできます。  
８．これをすべてのMDFファイルに対して行ってください  
  
## チュートリアル：ReportViewerレポートの作成  
ここでは下記のチュートリアルを行うための補足説明を行います。  
  
 __チュートリアル : ReportViewer レポートの作成__  
http://msdn.microsoft.com/ja-jp/library/ms252073%28v=vs.90%29.aspx  
このチュートリアルではテーブルを用いて一覧形式の帳票を作成します。  
  
SQLSERVER上のテーブルが最新のものと違うため、ドキュメント通りやっても動作しないので注意してください。  
また、チュートリアルでの言語はVB.NETですが、C#でも同じ手順で動作します。  
  
１．「WINDOWSフォームアプリケーション」を作成します  
２．「データソース接続」と「データテーブル」を定義します  
(1)ソリューションエクスプローラーから「新しい項目」の追加で「データセット」を追加します。  
https://qiita-image-store.s3.amazonaws.com/0/47856/d8dde476-66a5-7f8c-61b0-a3ade63b0486.png  
  
(2)データセットデザイナが表示されるので、ツールボックスの「TableAdapter」コントロールをドロップします。  
https://qiita-image-store.s3.amazonaws.com/0/47856/e50395a8-f40c-7ccf-05d2-77f0d1b00462.png  
これにより、TableAdapter構成ウィザードが開きます。  
  
(3)[データ接続の選択] ページで [新しい接続] をクリックします。  
  
(4)以下のような設定を行ってください  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/f0aa5b9b-81eb-01c1-885c-a2c93e4dee19.png  
  
※VisualStudio2008とSQLSERVER2012の場合はエラーが発生します。この時は付録の「VS2008からSQLSERVER2012に接続する」を参照してください。  
  
(5)接続を追加後、TableAdapter構成ウィザードの作成を進めていくとSQLステートメントの入力を求められます。  
この際、チュートリアル通りにやるとエラーになります。次のように修正してください。  
  
```sql
SELECT S.OrderDate,
 S.SalesOrderNumber,
 S.TotalDue AS TotalSales,
 C.FirstName,
 C.LastName
FROM HumanResources.Employee AS E INNER JOIN
 Person.Person AS C
ON E.BusinessEntityID = C.BusinessEntityID INNER JOIN
Sales.SalesOrderHeader AS S ON E.BusinessEntityID = S.SalesPersonID

```  
  
(6)これにより、データセットに次のようなデータが作成されました。  
https://qiita-image-store.s3.amazonaws.com/0/47856/c1391fc3-2506-81b2-2444-ab9772a7b249.png  
  
これはデータソースウィンドウから使用できます。  
https://qiita-image-store.s3.amazonaws.com/0/47856/54e62534-4cc2-4764-7173-51db952c3488.png  
  
  
３．新しいレポート定義ファイルを追加  
(1)「プロジェクト」メニューの「新しい項目」を選択します。  
  
(2)Repotingに存在する「レポート」を選択してください。  
https://qiita-image-store.s3.amazonaws.com/0/47856/b4af8209-395e-0a7e-e699-fbd51b029e09.png  
  
(3)[ファイル名] に「SalesOrders.rdlc」と入力し、[追加] をクリックすると、デザイン画面が開かれます。  
https://qiita-image-store.s3.amazonaws.com/0/47856/768dad03-7ba1-95cd-fbda-59fe07c9f693.png  
  
このようにGUIでレポートを編集するアプリケーションを「レポートデザイナ」といいます。レポート定義ファイルは「レポートデザイナ」だけでなく、下記のように「XML（テキストエディタ）」でも開くことができます  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/c2727927-4ee4-5515-75bb-c5c870f0b672.png  
  
４．レポートレイアウトでテーブルを追加する  
テーブルを利用して一覧形式の帳票を作成します。  
  
(1)レポートにテーブルを追加するために、ツールボックスの「テーブル」を選択後、「レポートデザイナ」をクリックします。  
これによって、「データセット」プロパティが起動するので、データソースにDataSet1を選択してOKボタンを押下してください。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/176bf4b0-c735-18d6-23d4-579d94f2fa2d.png  
  
以後、作成したテーブルとデータセットが関連づけられ、指定したデータセットのフィールドをテーブルのセルで使用することが可能になります。  
  
また、データセットプロパティを閉じると「レポートデザイナ」には３列のテーブルが作成されます。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/dc42b44f-e49b-161f-f37a-0d58979fd317.png  
  
(2)作成されたテーブルをクリックすると列ハンドルと行ハンドルが表示されます。  
https://qiita-image-store.s3.amazonaws.com/0/47856/e3dad1af-6fb5-263c-1698-c88ecfe5f0e3.png  
  
(3)左端の列ハンドルを右クリックして左側に列を挿入します。  
https://qiita-image-store.s3.amazonaws.com/0/47856/e879bb36-5d67-8fb5-c9ef-632cc48ef576.png  
  
(4)列の幅を調整します。「プロパティ」ウィンドウで「tablibn」というコントロールを選択して、「Size」ノードの「Width」に任意の数値を入力します。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/199a7957-1e70-6bf5-a84b-e22596d56321.png  
  
(5)テーブルに表示するデータを指定します。  
  
(ア)最初の列の中央行にマウスオーバをすると「データソース」のアイコンが出現するのでクリックをします。  
https://qiita-image-store.s3.amazonaws.com/0/47856/eb06f0a9-8797-cf93-2f72-bc1a673a4aa3.png  
  
(イ)関連付けるフィールドの一覧がメニューとして表示されるので「LastName」を選択してください。  
https://qiita-image-store.s3.amazonaws.com/0/47856/d261f7ef-e26d-1519-5fa1-756d8dd9ee40.png  
  
その結果、テーブルには以下のような値が書き込まれます。  
https://qiita-image-store.s3.amazonaws.com/0/47856/57abe1ce-055a-450f-356a-7d1678328e33.png  
  
１行目と２行目にどのような違いがあるのでしょうか？  
実は１行目はただの「文字」ですが、２行目は「式」になります。  
各セルを右クリックして「式…」を押下するとそのことが確認できます。  
  
１列目－１行目  
https://qiita-image-store.s3.amazonaws.com/0/47856/8e0bc030-48b3-1da1-b71e-4d761d3bca24.png  
  
１列目－２行目  
https://qiita-image-store.s3.amazonaws.com/0/47856/350608e9-5a47-f5b4-771f-c30e6e563874.png  
  
1行目は「Last Name」というただの文字で「＝」がなく式ではありません。  
２行目は「=Fields!LastName.Value」という式になっています。  
この式は指定したデータセットのフィールドLastNameの値をこのセルに表示することを意味します。  
  
(ウ)同様にOrderData,SalesOrderNumber、TotalSalesを２～４列目に指定します。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/f40fd32a-0427-1c77-9a46-3c1e0dc5510a.png  
  
これによりテーブルのすべてのセルに何らかの値が入力されました。  
  
５．フォームにReportViewerコントロールを追加する  
フォームにReportViewerコントロールを追加して、作成した帳票を表示するようにします。  
  
(1)フォームをデザイナで開いて、ツールボックスから「ReportViewerコントロール」を張り付けてください  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/f9a0282a-1ce4-a084-b7a6-12735de44644.png  
  
(2)ReportViewer　コントロールの「Dock」プロパティを「Fill」にすることでコントロールがFormの大きさに合わせて自動リサイズされるようになります。  
https://qiita-image-store.s3.amazonaws.com/0/47856/a97f4799-f74b-917e-0984-826a2fea849d.png  
  
  
(3)ReportViewerコントロールの右上隅の三角形をクリックします。  
https://qiita-image-store.s3.amazonaws.com/0/47856/60558e9a-07d6-a440-3548-1f13be874cc7.png  
  
これにより「ReportViewerタスク」メニューが表示されます。  
  
(4)レポートの選択でさっき作成したレポート定義ファイルを選択します。  
https://qiita-image-store.s3.amazonaws.com/0/47856/69e7c3f7-479e-e9ce-ebe5-79b37a64dc32.png  
  
これにより、Formには次のコントロールが追加されます。  
・DataSet  
・DataTableBindSource  
・DataTableAdapter  
  
これらのコントロールはレポート定義にデータベース上のデータを表示するために使用されます。  
  
(5)この時点でビルドして実行すると次のようなウィンドウが起動されます。  
https://qiita-image-store.s3.amazonaws.com/0/47856/23403d1c-7bde-32cd-e76e-93a8cc7b8eda.png  
  
### 主なアイコンの説明  
|アイコン|説明|  
|:-------|:---|  
https://qiita-image-store.s3.amazonaws.com/0/47856/2b33b35a-231e-731c-fb6c-2cef21a62a3a.png  
https://qiita-image-store.s3.amazonaws.com/0/47856/b083dcbe-fb49-c932-d4d0-7d743a0d2683.png  
https://qiita-image-store.s3.amazonaws.com/0/47856/76e5f6cf-011b-931b-7f6d-aa70b2b493ab.png  
https://qiita-image-store.s3.amazonaws.com/0/47856/ae9b2078-dae1-f61e-b09f-7dd96d68ceed.png  
https://qiita-image-store.s3.amazonaws.com/0/47856/edccf3f9-f868-01a7-7e6a-3355cab509cf.png  
https://qiita-image-store.s3.amazonaws.com/0/47856/5e802e50-7cd5-fc82-df40-f28acdce1cba.png  
  
### フィールドの書式を指定する方法  
レポートデザイナでは、フィールドの書式を設定することができます。  
まず、OrderDateの日付を日本語の書式に変更してみましょう。  
  
１.書式を指定したいセルを右クリックして「テキストボックスのプロパティ」を選択します。  
https://qiita-image-store.s3.amazonaws.com/0/47856/ecb11caf-1d23-b9c3-2f60-6dd53c885da0.png  
  
２.テキストボックスのプロパティダイアログでは「数値」項目を選択すると様々な書式を設定できます。  
カテゴリで「日付」を選択して書式を選択して、OKでダイアログを終了してください。  
https://qiita-image-store.s3.amazonaws.com/0/47856/39f41bbd-2b11-bb97-1e5d-e2d8263caf03.png  
  
３.ビルドして実行すると下記のようにOrderDateが選択した書式のものになっていることが確認できます。  
https://qiita-image-store.s3.amazonaws.com/0/47856/d85fc3e4-aef4-335e-3e12-8165bcdabe67.png  
  
４.同様にTotalSalesを選択して書式を「通貨」にします。  
https://qiita-image-store.s3.amazonaws.com/0/47856/3b6920d3-eefa-c2ec-1ae9-a435976ecf59.png  
小数点以下桁数や区切り記号の有無などを指定できます。  
  
５.これを再度ビルドして実行すると指定した書式でTotalSalesが表示されます。  
https://qiita-image-store.s3.amazonaws.com/0/47856/41666ad8-6608-2594-bc29-cbf136970554.png  
  
### セルの修飾  
各セルは、それぞれフォント、背景色、罫線などで修飾してデータを見やすくすることが可能です。  
１.修飾を行いたいセルを選択します。  
https://qiita-image-store.s3.amazonaws.com/0/47856/4f58f644-b795-b852-d00f-7eed0295085b.png  
  
２.この状態でプロパティのBackgroundColerで任意の背景色を選択します。  
https://qiita-image-store.s3.amazonaws.com/0/47856/278722c0-a5e4-a14e-fa6e-d63d44276da9.png  
これにより選択した範囲の背景色が変更されました。  
  
  
３.プロパティのColorに任意の値を指定すると文字の色が変更されます。  
https://qiita-image-store.s3.amazonaws.com/0/47856/dc3417df-e5c2-9151-b335-044be7c79831.png  
  
４.フォントについてはプロパティのFontを変更することで、フォントの種類やサイズなどが変更できます。  
https://qiita-image-store.s3.amazonaws.com/0/47856/937e233a-a2d4-adc3-47b2-f3e079896b62.png  
  
５.罫線はBorderStyleプロパティで変更できます。この例だと上下は実線、左右には線を引かない設定です。  
https://qiita-image-store.s3.amazonaws.com/0/47856/759c19fc-bfd6-ad3d-42c8-abb0cc055597.png  
  
６.ここまで変更したうえでビルドして実行すると、次のような表示になります。  
https://qiita-image-store.s3.amazonaws.com/0/47856/4b2b907d-0367-50f5-c156-7ce6078dac98.png  
  
### ヘッダーとフッターの設定  
印字した時のページ毎にヘッダーとフッターを指定できます。  
レポートデザイナを編集中に表示される「レポート」メニューより「ページヘッダーの追加」「ページフッターの追加」を選択してください。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/83283c82-d72f-55f4-c4ef-3dfe9db504e4.png  
https://qiita-image-store.s3.amazonaws.com/0/47856/144d2e02-f8c2-e144-e2d6-883629aaac9b.png  
  
これにより、ページヘッダーとページフッターを表示できるようになりました。  
ヘッダー、フッターの高さを変更するにはヘッダー、フッターのプロパティのHeightで変更ができます。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/3a5f5ad7-f3a5-d11e-0457-f5f7348a9a94.png  
  
ヘッダー、フッターには下記のコントロールのみが配置可能です。テーブルやマトリックスといったコントロールは配置できません。  
  
 __ヘッダー、フッターに配置可能なコントロール__  
・テキストボックス  
・線  
・四角形  
・画像  
  
#### ヘッダー、フッターに固定の文字を表示  
この例ではヘッダーに帳票名を表示します。ヘッダーにツールボックスよりテキストボックスを選択してヘッダーに張り付けます。貼り付けたテキストボックスに任意の文字を入れてください。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/d38e3356-fb72-6aa7-6e64-3b73f4371602.png  
  
テキストボックスの枠線が表示されていますが、実際、実行すると非表示になっています。そして、複数ページにわたり、入力した文字が出力されていることを確認できます。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/ef677fe2-5eca-fb60-e400-96dc113eb3dd.png  
  
#### ヘッダー、フッターに現在の日付を表示する  
テキストボックスには固定の文字だけでなく式を入力することで、日付などを表示することが可能です。固定文字の時と同様にヘッダーまたはフッターにテキストボックスを張り付けます。その後、右クリックでメニューを表示して「式」を選択します。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/0ab85c6f-5971-fcf2-c962-49e67b4e20b3.png  
  
「式」ダイアログを表示さるので、ここに式を入力することが可能です。式を直接入力することも可能ですし、「カテゴリ」一覧から使用できる数式を選択ですることも可能です。  
今日の日付を表示するには「カテゴリ」の「共通の関数→日付と時刻」を選択し、アイテムで「Today」を選択します。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/35f47d38-be84-b642-669c-3d1b7b79ef22.png  
  
この時点で表示をすると「2013/12/16 0:00:00」というような書式になります。  
https://qiita-image-store.s3.amazonaws.com/0/47856/ed7c514e-a891-c7f4-0e53-d1549d6d572c.png  
  
テキストボックスの書式は、テキストボックスのプロパティから変更が可能です。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/92208ded-862c-464c-cbeb-5288d828dfc4.png  
  
これにより以下のように適切な書式の日付が表示されます。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/93fc03f0-9cfb-5a0c-c372-3195756af4d5.png  
  
  
#### ヘッダー、フッターにページ数  
日付と同様に、テキストボックスの式を用いてページ数を表示することが可能です。  
ヘッダーにテキストボックスを張り付けて以下の式を入力します。  
  
```csharp
=Globals!PageNumber & "/" & Globals!TotalPages
```  
  
式で文字として連結を行う場合は「＆」演算子を使用します。  
これにより、ヘッダーは次のように、「現在のページ数 / 総ページ」を表示します。  
  
式には様々な機能が存在しており、次のページから確認することが可能です。  
  
 __レポートの一般的な式 (Visual Studio レポート デザイナ)__  
http://msdn.microsoft.com/ja-jp/library/ms251668%28v=vs.90%29.aspx  
  
  
 __式のリファレンス (レポート ビルダーおよび SSRS)__  
http://msdn.microsoft.com/ja-jp/library/ee240847.aspx  
  
開発者は独自の数式を作成することも可能で、その方法は「カスタムコードの作成」にて説明します。  
  
### 各ページにテーブルのヘッダーを表示する  
ヘッダーとフッターの説明を行いました。しかし、説明した方法では大量のデータを保持するテーブルのヘッダーを表示ができません  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/4605414d-99ea-efd7-af0a-3cadad80228d.png  
  
テーブルのヘッダーをページ毎に表示するには、レポートデザイナの行グループのプロパティを修正する必要があります。そのために、まず、行、列グループの「詳細設定モード」にチェックをつけます。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/58f9f993-929c-19c2-0847-9ee8bc366c66.png  
  
詳細設定モードをチェックすることで、行グループ、列グループに「(静的)」というアイテムが表示されます。各ページでテーブルの行のヘッダーを繰り返し表示するには、行グループの「(静的)」を選択して、プロパティの「RepeatOnNewPage」をTrueにしてください。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/c9f82e99-55d4-4615-d19c-df67beb9d7f3.png  
  
これにより、複数ページにわたりテーブルのヘッダーを表示することができます。  
https://qiita-image-store.s3.amazonaws.com/0/47856/11faa4f3-49c3-4c0e-81c8-3ed677bc8522.png  
  
### テーブルのグループを定義する方法  
ReportViewerコントロールのテーブルでは、与えられたデータをグループ化して表示することができます。今回の例では、LastNameとFirstNameでデータをグループ化して各個人の売り上げの帳票を出力します。  
  
１．行ハンドルを右クリックして「グループの挿入」＞「親グループ」を選択します。  
https://qiita-image-store.s3.amazonaws.com/0/47856/947dc0e7-98d3-f9e7-c9a2-bcccd045d731.png  
  
２．グループダイアログが表示されるので、グループ化のコンボボックスで「LastName」を選択し、「グループヘッダーの追加」と「グループフッターの追加」にチェックをします。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/88922af3-f45b-971d-6c92-1660d6fd938a.png  
  
これより、テーブルにはグループの名称を表示する列と、行グループにグループが追加されました。  
https://qiita-image-store.s3.amazonaws.com/0/47856/b6351e4f-dedb-9359-f3e4-e0030f5afaad.png  
  
３．この例ではグループ化の条件が[LastName]だけです。これでは同じLastNameを持つ別の人物も同じグループにしてしまいます。そのため、グループ化の条件に「FirstName」も追加する必要があります。  
まず、行グループで条件を追加したいグループを選択して右クリックして「グループプロパティ」を選択します。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/3e7c68a5-a20e-d43a-7313-e84c696e65cb.png  
  
グループプロパティ」ダイアログではグループ化についてのプロパティを表示確認できます  
https://qiita-image-store.s3.amazonaws.com/0/47856/00c7c372-8066-6d56-aab5-24d7a0836ffd.png  
  
条件を追加するには、グループ化の追加ボタンを押して条件に「FirstName」を追加します。  
https://qiita-image-store.s3.amazonaws.com/0/47856/c1faafef-137a-e108-a0b1-4703610bc64d.png  
  
４．この状態でレポートを出力すると次のようになります。  
https://qiita-image-store.s3.amazonaws.com/0/47856/c6b60d05-32ff-0645-ab4e-60b18fbed39f.png  
  
各グループの先頭と、末尾にグループヘッダーとグループフッターが付与されていることが確認できます  
  
５．この帳票ではグループ化を行う際に、テーブルに列が自動的に挿入されましたが、これは表示に使用するだけなので、削除してもグループ化は解除されません。  
テーブルで挿入された列を右クリックして、「列の削除」を選択します。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/4aa0de52-1fab-659d-0b36-8eaee995f083.png  
  
この状態でレポートを作成すると次のように、グループ用の列が存在しなくても、グループ化されていることがわかります。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/6b195cb5-26e7-7283-04cf-40d768a10d9d.png  
  
#### グループの削除方法  
グループを削除するには、削除したい行グループを選択して、右クリックで「グループの削除」を行います。  
https://qiita-image-store.s3.amazonaws.com/0/47856/e86a0676-5f5f-e92c-ef67-59b40ba2bd7a.png  
  
#### グループ別のデータ集計  
グループ化を行うことで、グループ別にデータを集計することができます。今回はセールスマン毎での売り上げの合計を求めます。  
  
１．グループヘッダーで合計を出力したい箇所を右クリックして「式」を選択します。  
https://qiita-image-store.s3.amazonaws.com/0/47856/e45731c6-f7ff-1770-19ca-13284ac37fa0.png  
  
２．式ダイアログが表示されるので、「式の設定」に次の式を入力します。  
  
```csharp
=Sum(Fields!TotalSales.Value)
```  
  
３．追加を行ったら、このセルに対して通貨型の書式を設定します。「フィールドの書式を指定する」を参考にしてください。  
  
４．これを実行することで、各グループのヘッダーに売り上げの総計が表示されるようになります。  
https://qiita-image-store.s3.amazonaws.com/0/47856/8601efdb-2f20-68b5-93e3-3ed9724455b1.png  
  
  
  
次にグループヘッダーにフルネームとデータ件数を表示します。  
  
１．テーブルの１行目、１列目に次の式を入力します。  
  
```csharp
=Fields!FirstName.Value + " " + Fields!LastName.Value + ": " + vbCrLf + Count(Fields!SalesOrderNumber.Value).ToString()
```  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/879341cc-b6a2-3425-e72f-c8f3350c812c.png  
  
２．「LastName」が行毎に表示されているので、それを削除します。これにより、名前が行ごとにでるのではなく、グループヘッダーのみに表示されます。  
https://qiita-image-store.s3.amazonaws.com/0/47856/07a16daa-8695-fb2c-6ce8-e5dc03ea3c1e.png  
  
３．この状態でレポートを出力することでグループヘッダーにフルネームとデータ件数が表示されます。  
https://qiita-image-store.s3.amazonaws.com/0/47856/41547adc-5a97-a482-9398-fac48f7fcc2c.png  
  
４．レポートで、次のページをみてみます。以下のように先ほど設定したはずのテーブルのヘッダーが表示されなくなっています。  
https://qiita-image-store.s3.amazonaws.com/0/47856/680fbccb-50dc-6c66-6eb2-92c9eff7ecb6.png  
これはグループ化を行ったため、行グループの状態が、さっきと変更されたために発生します。  
  
５．行グループの一番上の「静的」のプロパティで「RepeatOnNewPage」を「True」、「KeepWIthGroup」を「After」にしてください。  
https://qiita-image-store.s3.amazonaws.com/0/47856/b0b91427-7ebf-66cc-e2df-ee019a99773e.png  
  
６．その後、実行すると次ページ以降にもテーブルのヘッダーが表示されます。  
https://qiita-image-store.s3.amazonaws.com/0/47856/3e9f60e4-b203-ca19-9eb5-1cad43bfddcc.png  
  
７．次にグループが複数ページにまたがった場合に、グループヘッダーを同様にページの先頭に表示するようにします。行グループで、グループ名の直下の「静的」を選択して同様にプロパティの「RepeatOnNewPage」を「True」、「KeepWIthGroup」を「After」にしてください。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/4f4125e6-61b3-c3c3-35dd-4ce45eac278b.png  
  
８．これにより、各ページの先頭にグループのヘッダーが表示されるようになります。  
https://qiita-image-store.s3.amazonaws.com/0/47856/0ba185c6-a2ac-3027-856e-45ff19ee45b3.png  
  
#### 表形式のレポート内のグループを並べ替える  
ReportViewerでは並び替えの機能を提供しています。これにより、作成したグループを並べ替えることができます。この例ではグループが保持するデータの数順に並び替えます。  
  
１．行グループの最上位のアイテムを選択して右クリックして、「グループプロパティ」を選択します。  
https://qiita-image-store.s3.amazonaws.com/0/47856/33f252a8-4aa3-6526-986c-49e8b97c9f59.png  
  
２．「グループプロパティ」ダイアログが開くので「並べ替え」を選択して「追加」ボタンをおします。  
https://qiita-image-store.s3.amazonaws.com/0/47856/d35eb758-a121-6897-23bb-5b0bf9c5d95d.png  
  
並び替えの条件には以下の式、順序は「昇順」としてOKを押してください。  
  
```csharp4=Count(Fields!SalesOrderNumber.Value)
```  
  
３．ここで実行を行うと販売数の少ない人ほど最初のページに表示されるようになります。  
  
#### 表形式のレポートのグループ内の詳細行を並べ替える  
１.行グループで、グループ内の詳細を選択して「グループプロパティ」を選択します。  
https://qiita-image-store.s3.amazonaws.com/0/47856/de623a53-da2c-d4c8-1a9a-bbe4d2a44b57.png  
  
２.グループプロパティダイアログで、「並び替え」に縦棒を追加します。  
[TotalSales]をコンボボックスで選択して、順序を「降順」にしてください。  
https://qiita-image-store.s3.amazonaws.com/0/47856/9799fb2f-c25d-2154-2bb2-b92186b1956d.png  
  
３.ここで実行すると、売り上げの降順でグループ内が並び替えされているようになります。  
https://qiita-image-store.s3.amazonaws.com/0/47856/09caafae-2f22-00a8-aad8-9b92399669d7.png  
  
## マトリックスを用いたクロス集計  
この項目ではマトリックスを用いたクロス集計を行います。クロス集計とは項目を掛け合わせて、それぞれの項目が交わるセルに合計値や数などを記載した表です。  
このチュートリアルは下記のMicrosoftのチュートリアルを説明しています。  
  
 __チュートリアル : ローカル処理モードでのデータベース データ ソースと ReportViewer Windows フォーム コントロールの使用__  
http://msdn.microsoft.com/ja-jp/library/ms251724%28v=vs.90%29.aspx  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/c615ac95-a7be-1d98-6d50-20f8f7a1b944.png  
  
１．データソース接続とDATATABLE定義  
  
(1)[プロジェクト]メニューの[新しい項目の追加]をクリックします  
  
(2)[新しい項目の追加]ダイアログで[データセット]を選択して追加します。この際、任意の名称を入力できます。ここでは規定の名前であるDataSet1.xsdを使用します  
  
(3)ツールボックスから[TableAdapter]をデータセットデザイン画面にドラッグします。これによりウィザード画面が表示されます。先のチュートリアルで実行した『「データソース接続」と「データテーブル」を定義します』を参照してAdventureWorks2012に接続をします。  
  
(4)SQLステートメントの入力まで進めて次のSQLを入力します。  
  
```sql
SELECT d.name as Dept, s.Name as Shift, e.BusinessEntityID as EmployeeID
FROM (HumanResources.Department d
INNER JOIN HumanResources.EmployeeDepartmentHistory e
ON d.DepartmentID = e.DepartmentID)
INNER JOIN HumanResources.Shift s
ON e.ShiftID = s.ShiftID
```  
  
２．レポートのデザイン  
  
(1)[プロジェクト]メニューの[新しい項目の追加]をクリックします  
  
(2)Reportingの[レポート]を選択します。これでレポートデザイナが起動します。  
  
(3)ツールボックスのマトリックスを選択してレポートデザイナにドロップしてください。  
https://qiita-image-store.s3.amazonaws.com/0/47856/3a43ab93-45af-9e67-0207-b6d844abaa05.png  
  
(4)データセットのプロパティが起動するので、先ほど追加したDataSetを選択してください。  
https://qiita-image-store.s3.amazonaws.com/0/47856/08bb812a-8665-8a89-f893-0add5a0a06e0.png  
  
これでOKを押すことでマトリックスがレポートに追加されます。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/82a9e21f-a284-7bb3-478c-7acc9bcef194.png  
  
(5)「行」と記述しているセルに部署を表すDeptを選択します。  
https://qiita-image-store.s3.amazonaws.com/0/47856/f54881f0-9b57-c97a-6410-4d5b8e4b3cd7.png  
  
(6)「列」と記述しているセルにシフトを表すShiftを追加します  
https://qiita-image-store.s3.amazonaws.com/0/47856/2a1553e7-e345-6402-98c9-0530f491bdff.png  
  
(7)データ」とあるセルを右クリックして式プロパティダイアログを表示して式を入力します。  
https://qiita-image-store.s3.amazonaws.com/0/47856/f7a9f0b8-5ff5-7b15-1b0b-93d1a7c245a1.png  
  
```csharp
=Count(Fields!EmployeeID.Value)
```  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/410936d8-eb1c-4568-3040-3199dca147a8.png  
  
３．アプリケーションへのReportViewerコントロールの追加  
(1)FormデザイナにおいてツールボックスのReportViewerコントロールをFormに張り付けます。  
https://qiita-image-store.s3.amazonaws.com/0/47856/417dc1d9-4c4e-2dc2-ff3b-8683b253390a.png  
  
(2)ReportViewerコントロールのプロパティの[Dock]を[Fill]に指定します。これでReportViewerコントロールはForm大きさに一致するようになります。  
  
(3)右上隅の三角形をクリックしてReportViewerタスクメニューを表示して、レポート選択コンボボックスで作成したレポートを選択します。  
https://qiita-image-store.s3.amazonaws.com/0/47856/d779411b-5151-f72e-686d-e1b9fa977369.png  
  
(4)指定したレポートにデータを表示するためにDataSetコントロール、DataTableBindingSourceコントロール、DataTableAdapterコントロールが追加されます。  
https://qiita-image-store.s3.amazonaws.com/0/47856/4f505a39-f66b-d0ee-3c0e-ad9d35212edf.png  
  
(5)この時点でビルドして実行すると次のようなクロス集計を行うレポートが作成されます。  
https://qiita-image-store.s3.amazonaws.com/0/47856/4f948938-99aa-f2d8-d8d0-4dc992aac660.png  
  
## 単票形式の帳票の作成  
このチュートリアルでは一覧形式では表現できない帳票を作成します。  
この例では以下のようなユーザー情報を名刺として印字するような帳票を作成します。  
この例では画像データを使用するので、下記からダウンロードするか、ご自分でPNGファイルをご用意してください。  
http://needtec.sakura.ne.jp/doc/ReportCard.zip  
  
### データソースとして使用するビジネスオブジェクトの作成  
今回の使用するデータはデータベースではなく、クラスで定義した情報を使用します。  
Users.csというクラスを追加して以下のような実装をします。  
  
```csharp
using System;
using System.Collections.Generic;
using System.Drawing;

class User
{
 public string Name { get; set; }
 public DateTime Birth { get; set; }
 public string Tel { get; set; }
 public string Address { get; set; }
 public Byte[] ImageData { get; set; }

 public User(string name, string birth, string tel, string address, string imagepath)
 {
 Name = name;
 try
 {
 Birth = DateTime.Parse(birth);
 }
 catch (Exception ex)
 {
 throw new ArgumentException("birthの書式が不正です\n" + ex.Message);
 }
 Tel = tel;
 Address = address;
 try
 {
 Image img = Image.FromFile(imagepath);
 ImageConverter imgcv = new ImageConverter();
 ImageData = (byte[])imgcv.ConvertTo(img, typeof(byte[]));
 }
 catch (Exception ex)
 {
 throw new ArgumentException("imagepathの指定が不正です\n" + ex.Message);
 }


 }
}
class Users
{
 private List<User> users;
 public Users()
 {
 users = new List<User>();
 users.Add(new User("やる夫", "2005/1/1", "000-000-1111", "2ch ニュー即 VIPスレ 000-333-555 紙スレ", @"..\..\Data\img001.png"));
 users.Add(new User("やらない夫", "2005/2/1", "000-000-1112", "2ch VIPスレ", @"..\..\Data\img002.png"));
 users.Add(new User("ゆっくり霊夢", "2008/1/1", "000-001-1111", "ニコニコ　実況", @"..\..\Data\img003.png"));
 users.Add(new User("ゆっくり魔理沙", "2008/2/1", "000-001-1112", "ニコニコ　実況", @"..\..\Data\img004.png"));
 users.Add(new User("はちゅねミク", "2009/3/1", "000-001-1113", "ニコニコ　実況", @"..\..\Data\img005.png"));

 }
 public List<User> GetUsers()
 {
 return users;
 }
}
```  
  
### レポート定義にオブジェクトを関連づける  
ここではレポート定義ファイルを作成してデータソースとしてオブジェクトを関連付ける方法を記述します。  
  
１．プロジェクトメニューの「新しい項目の追加」でレポートを追加してください。  
  
２．レポートのデザイナを選択中に「表示」メニューから「レポートデータ」を選択することで「レポートデータ」ウィンドウが表示されます。これはCTRL＋ALT＋Dで代替できます。  
  
３．レポートデータウィンドウにて、「データセット」を右クリックして「データセットの追加」を選択してください。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/3329f4dc-3d5b-7595-f69a-6e8e189904f8.png  
  
４．データソース構成ウィザードが起動するので、ウィザードを利用してデータソースを作成します。  
まずオブジェクトを選択して「次へ」を押します  
https://qiita-image-store.s3.amazonaws.com/0/47856/9ed1d337-1b1d-a526-f9e5-b94c81f21614.png  
  
オブジェクトの一覧が表示されるので作成したUserオブジェクトが表示されるまで展開をしてUserオブジェクトにチェックをつけ「完了」します。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/3505e106-ff51-725d-0071-e42f627af53c.png  
  
５．データセットプロパティ　ダイアログでデータセットが作成されるのでOKを押します。  
https://qiita-image-store.s3.amazonaws.com/0/47856/c5160fcb-b44b-4ac4-ee61-4906526430a1.png  
  
以降、レポート定義ではデータベースをデータソースとした時と同様に、データを用いてレポートを作成できます。  
  
### レポートの大きさを名刺サイズにする  
レポートの大きさを9.1cm x 5.5cmの名刺サイズに変更します。  
  
１．レポートデザイナで灰色の余白部分を右クリックして「レポートのプロパティ」を選択します。  
https://qiita-image-store.s3.amazonaws.com/0/47856/694076a7-0645-2fb3-0232-97b1effce30b.png  
  
２．ページ設定で用紙のサイズを「カスタム」にして「幅」9.1cm、「高さ」5.5cmとします。また、余白はすべて0cmとします。  
https://qiita-image-store.s3.amazonaws.com/0/47856/3acde418-2b59-f5dd-eea9-91a4aafdb11c.png  
  
３．レポートデザイナで本文を選択してプロパティのSizeで用紙の大きさを指定します。用紙と同じサイズだとはみ出て、改行してしまうので、２，３ｍｍ程度少なく設定します。  
https://qiita-image-store.s3.amazonaws.com/0/47856/d5c832a5-3963-f47d-e032-87ecb01faee5.png  
  
#### 一覧データによる繰り返しデータの自由形式でのレイアウト  
ここでは一覧データを用いて、繰り返しデータを一覧表という形式ではなく自由なレイアウトでデザインします。  
  
１．レポートデザイナにツールボックスから「一覧」を選択してドロップします。この際、作成された一覧の大きさを本文いっぱいに広げてください。  
https://qiita-image-store.s3.amazonaws.com/0/47856/308c94eb-b9d7-484d-663f-01699d637ee8.png  
  
２．名前を表示するテキストボックスを追加します。  
データのアイコンをクリックして「Name」を選択してください。  
https://qiita-image-store.s3.amazonaws.com/0/47856/303ab7d8-7001-6a13-ff1b-a1110dbf8b4b.png  
  
表示するデータを指定したら、レイアウトを指定します。テキストボックスのプロパティを開いて配置で縦、横を[中央揃え]にします。  
https://qiita-image-store.s3.amazonaws.com/0/47856/61582a68-a749-f012-ed17-80faefe8e10e.png  
  
次にフォントを指定します。MS Pゴシック、サイズを20pt、太字にチェックをつけます。  
https://qiita-image-store.s3.amazonaws.com/0/47856/05fb06ec-2c91-b726-8330-ea8f446f5ad3.png  
  
３．Nameと同様に、Address、Tel、Birthに関してテキストボックスで表示します。Birthに関してはテキストボックスのプロパティの「数値」で書式を設定して、日付型の書式にします。  
https://qiita-image-store.s3.amazonaws.com/0/47856/a786a324-1f8e-373e-bc4a-954086ce0fff.png  
  
４．プロフィール画像を表示するために画像をドロップします。  
https://qiita-image-store.s3.amazonaws.com/0/47856/c8c90de5-39d1-7530-aae9-12e3c3b7a0d3.png  
  
ドロップをすると画像のプロパティが表示されるので、画像ソースの選択で「データベース」、次のフィールドを適用で「[ImageData]」を選択して、MIMEの種類は「image/png」にします。  
https://qiita-image-store.s3.amazonaws.com/0/47856/c80d2591-99af-deaa-b55d-a1d1e51b06a3.png  
  
ここでOKを押すことにより、画像が表示されるようになります。  
https://qiita-image-store.s3.amazonaws.com/0/47856/0007148f-ef14-0dc7-29e9-2818c15a89d8.png  
  
### アプリケーションへのREPORTVIEWERコントロールの追加  
１.FormデザイナにおいてツールボックスのReportViewerコントロールをFormに張り付けます。  
  
２.ReportViewerコントロールのプロパティの[Dock]を[Fill]に指定します。これでReportViewerコントロールはForm大きさに一致するようになります。  
  
３.右上隅の三角形をクリックしてReportViewerタスクメニューを表示して、レポート選択コンボボックスで作成したレポートを選択します。  
指定したレポートにデータを表示するためにUserBindSourceコントロールが追加されます。  
https://qiita-image-store.s3.amazonaws.com/0/47856/27c1a068-d113-bb7f-a152-8413eee601ea.png  
  
４.Form_LoadイベントでRefreshReportを行う前に、UserBindSourceにデータを追加します。  
  
```csharp
private void Form1_Load(object sender, EventArgs e)
{
 Users users = new Users();
 UserBindingSource.DataSource = users.GetUsers();
 this.reportViewer1.RefreshReport();
}
```  
  
５.この状態でビルドを実行して印刷プレビューを見ると以下のようになります。  
https://qiita-image-store.s3.amazonaws.com/0/47856/ebb75ff9-8e27-2c15-0415-a2e030090467.png  
  
このようにReportViewrを使用することで単票形式の帳票が容易に作成できることが確認できました。  
  
  
## プレビューを使用しないローカルレポートの印字  
ReportViewerコントロールでプレビューを表示しないで直接印字するには、ReportViewコントロールのLocalReportを画像にExportしてそのデータをプリンタに送ります。  
この方法は、以下に記述してあります。  
  
 __チュートリアル : プレビューを使用しないローカル レポートの印刷__  
http://msdn.microsoft.com/ja-jp/library/ms252091%28v=vs.90%29.aspx  
  
このチュートリアルを実行するまえに次のページのページを参照して、Report.rdlcファイルとData.xmlを作成してください。  
  
 __印刷チュートリアルのサンプル データおよびレポート__  
http://msdn.microsoft.com/ja-jp/library/ms251734%28v=vs.90%29.aspx  
  
## レポートパラメータの使用方法  
レポート定義ファイルでレポートパラメータを使用することで、実行時に任意の値を与えて、それをレポート中に表示することができます。  
  
１.レポートのデザイナを選択中に「表示」メニューから「レポートデータ」を選択することで「レポートデータ」ウィンドウが表示されます。これはCTRL＋ALT＋Dで代替できます。  
  
２.レポートデータウィンドウで「パラメーター」を選択して右クリックを押し、「パラメーターの追加」を押下します。  
https://qiita-image-store.s3.amazonaws.com/0/47856/b8909eb9-ee09-49b8-d069-7a9debbc582c.png  
  
３.「レポート パラメーターのプロパティ」ダイアログで任意の名称を付与します。  
https://qiita-image-store.s3.amazonaws.com/0/47856/e823c567-2965-8314-c8d8-cf32c012f41e.png  
  
４.以後、このレポート定義ファイルでは式に追加したパラメーターが使用できます。以下のように式ダイアログのカテゴリ「パラメーター」に今追加したパラメーターが増えているのを確認できます。  
https://qiita-image-store.s3.amazonaws.com/0/47856/dffb6a76-8985-5ae3-6ebc-6cbbd3fc9c79.png  
  
この例では帳票の左上にレポートパラメータを表示するテキストボックスを追加します。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/0e087569-e322-1a46-5add-2eca0720024d.png  
  
５.レポートコントロールが表示される前に、レポートパラメータを以下のようにして与えます。  
  
```csharp
private void Form1_Load(object sender, EventArgs e)
{
 this.DataTable1TableAdapter.Fill(this.DataSet1.DataTable1);
 // レポートパラメータを作成
 ReportParameter param = new ReportParameter("ReportParameter1","レポート変数");
 reportViewer1.LocalReport.SetParameters(param);
 // レポート表示
 this.reportViewer1.RefreshReport();
}
```  
  
６.実行をするとプログラムで与えた文字がレポートに表示されます。  
https://qiita-image-store.s3.amazonaws.com/0/47856/b05ed397-42dd-98f1-702c-e53c31a82f5c.png  
  
## カスタムコードの作成  
カスタムコードを用いることで、独自の数式をReportViewer内で使用できます。カスタムコードの書き方は２種類存在しており、レポート定義ファイルにコードを組み込む方法と、グローバルアセンブリを使用する方法があります。  
  
### レポート定義ファイルにコードを組み込む  
１．レポートデザイナ使用中に「レポート」メニューの「レポートのプロパティ」を選択して「レポートのプロパティ」を表示します。  
  
２．コードを選択してカスタムコードに任意のコードを入力します。  
https://qiita-image-store.s3.amazonaws.com/0/47856/91c30588-f799-6e44-4d17-0539718b94e2.png  
  
このコードは指定した文字列中にBikeという文字があったら、Bicycleに変更します。  
  
```vbnet
Public Function ChangeWord(ByVal s As String) As String
 Dim strBuilder As New System.Text.StringBuilder(s)
 If s.Contains("Bike") Then
 strBuilder.Replace("Bike", "Bicycle")
 Return strBuilder.ToString()
 Else : Return s
 End If
End Function
```  
３．レポート定義にテキストボックスを追加して式に数式を入力します。  
https://qiita-image-store.s3.amazonaws.com/0/47856/5c189fcd-bdf9-f8a8-2321-02c7df7b37f7.png  
  
```csharp
=Code.ChangeWord("TEST Bike da")
```  
  
なお、インテリセンスは効きません  
  
４．この状態でレポートを表示すると次のようになります。  
https://qiita-image-store.s3.amazonaws.com/0/47856/31cea945-60e5-e419-7a2a-d974d7df0d38.png  
  
  
### グローバルキャッシュアセンブリを使用したカスタムコード  
VB.NETなどでクラスモジュールを作成して、それをグローバルキャッシュアセンブリとして登録することで、ReportViewerはそのカスタムコードを利用できます。  
  
.NET 4.0より前では以下のCustom Assembliesのやり方で作成できるはずです。  
http://www.codeproject.com/Articles/38554/Microsoft-Reporting-Services-Part-II  
  
.NET 4.0以降において、このやり方は通用しません。  
以降、.NET4.0以降でカスタムアセンブリを追加する方法を紹介します。  
  
#### 厳密な名前を持つクラスライブラリを作成する  
１．クラスライブラリを作成します。今回は「ReportViewerCustomAsmTest」という名称で作成します。  
https://qiita-image-store.s3.amazonaws.com/0/47856/120035a1-bb5d-8ade-080a-22088ceafb20.png  
  
２．追加した「ReportViewerCustomAsmTest」のプロジェクトのメニューで「ReportViewerCustomAsmTestのプロパティ」を選択してください。  
  
３．署名タブにて「アセンブリに署名する」をチェックして「厳密な名前のキーファイルを選択してください」で＜新規作成＞を選択します。  
https://qiita-image-store.s3.amazonaws.com/0/47856/b5cc625a-2fbb-5dc1-2a09-0a3942fdfa18.png  
  
４．「厳密な名前キーの作成」ダイアログが表示されるのでキーファイル名とパスワードを入力してOKを押してください。  
https://qiita-image-store.s3.amazonaws.com/0/47856/6716f280-2f9d-ebd2-13be-626fde552b62.png  
  
５．デフォルトで作成されたCustomCode.csを次のように置き換えます。  
  
```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace ReportViewerCustomAsmTest
{
 public class CustomCode
 {
 public string CutText(string txt)
 {
 if (txt.Length >= 30)
 {
 return txt.Substring(0, 26) + "...";
 }
 else
 {
 return txt;
 }
 }
 public static String SharedCutText(string txt)
 {
 if(txt.Length >= 30 )
 {
 return txt.Substring(0,26) + "...";
 }
 else
 {
 return txt;
 }
 }
 }
}
```  
  
５．AllowPartiallyTrustedCallers属性を指定します。  
C#の場合は、AssemblyInfo.csに下記を追記してください。  
  
```csharp
[assembly: System.Security.AllowPartiallyTrustedCallers()]
```  
  
もしVB.NETで作成していた場合、AssemblyInfo.vbはデフォルトで表示されていません。表示するにはソリューションエクスプローラーで「すべてのファイルを表示」を押してください。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/57501ced-c047-f45b-841b-d3cf48b36c3f.png  
  
これによりAssemblyInfo.vbが表示されるので、次を追記してください。  
  
```vbnet
<Assembly: System.Security.AllowPartiallyTrustedCallers()>
```  
  
６．ビルドしてクラスライブラリを作成します  
  
７．「開発者コマンドプロンプト for Visual Studio 2013」を管理者モードで起動します。  
このツールはVisualStudioをインストールすると次のフォルダに作成されます。  
  
```
C:\Program Files (x86)\Microsoft Visual Studio 12.0\Common7\Tools\Shortcuts
```  
  
８．コマンドプロンプト上で下記のコマンドを実行してグローバルキャッシュアセンブリとして登録します。  
  
```
gacutil /i ReportViewerCustomAsmTest.dll
```  
  
ただしく実行されると次のようなメニューが表示されます。  
  
```
Microsoft (R) .NET Global Assembly Cache Utility. Version 4.0.30319.33440
Copyright (c) Microsoft Corporation. All rights reserved.

アセンブリが正しくキャッシュに追加されました
```  
  
#### グローバルキャッシュアセンブリを削除するには？  
アセンブリ名（ファイル名ではない）を指定して/uオプションをつけてgacutilを実行してください。  
  
```
gacutil /u ReportViewerCustomAsmTest
```  
  
#### グローバルキャッシュされたアセンブリをレポートで使用する方法  
１．レポートデザイナを編集中に「レポート」メニューより「レポートのプロパティ」を選択します。  
  
２．「レポートのプロパティ」ダイアログの参照タブを選択して、アセンブリを追加します。  
https://qiita-image-store.s3.amazonaws.com/0/47856/688af9bd-5cda-90f1-fc24-33cb2e060c8e.png  
  
[…]ボタンを押すと、「参照の追加」ダイアログが開くので作成したDLLを選択してOKボタンを押してください。  
https://qiita-image-store.s3.amazonaws.com/0/47856/727803ff-4b54-af0d-7700-6ba611b799a8.png  
  
参照対象のアセンブリが追加されます。  
https://qiita-image-store.s3.amazonaws.com/0/47856/18e70236-35d0-66a2-80da-8837b44631c1.png  
  
３．スタティックなメソッドを使用するだけならこれでいいのですが、もし、インスタンスを作成する必要がある場合は、クラス名の追加をする必要があります。  
今回は以下のような内容を追加します。  
  
クラス名：ReportViewerCustomAsmTest.CustomCode  
インスタンス名：myClass  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/d50333bb-b106-7203-834c-4ae78d335191.png  
  
４．レポート定義でテキストボックスに式を追加します。  
スタティックなメソッドを利用する例：  
  
```csharp
=ReportViewerCustomAsmTest.CustomCode.SharedCutText("tttttttttttttttttttttttttttttttttttttttt")
```  
  
インスタンスメソッドを利用する例：  
  
```csharp
=Code.myClass.CutText("dddddddddddddddddddddddddddasdfasdfadfd")
```  
  
インスタンスを使う場合はプロパティで追加したインスタンス名を使用する必要があることがわかります。なお、やはりインテリセンスはききません。  
  
５．この状態で実行することで、カスタムコードが実行されていることが確認できます。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/f6d467a1-3798-43b9-4d71-209104380dbd.png  
  
## 改ページのTips  
ここでは改ページに関するTipsを紹介します。  
  
### 特定の行数毎ごとに改行をする  
ここでは特定の行数毎に改行する方法を紹介します。  
  
１．「チュートリアル：ReportViewerレポートの作成」と同じ手順でデータセットを作成します。  
  
２．レポートデザイナでテーブルを追加して以下のような一覧を作成します。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/3b15bb0f-3156-0076-82ce-89a99c543181.png  
  
３．行グループでグループを追加します。  
https://qiita-image-store.s3.amazonaws.com/0/47856/221c1fc0-084f-534d-2bfc-202c0be29b44.png  
  
グループ化をする際の条件にRowNumberを用いて行数でグループ化するようにします。この例では20行おきでグループかします。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/aba99fff-54d4-5de1-b952-139333bdced7.png  
  
```csharp
=CEILING(RowNumber(Nothing)/20)
```  
  
４．グループのプロパティで「並び替え」タブを選択して「式」を削除します。  
グループ化の条件と並び替えの条件の式は先に入力したものと同じになります。しかし、並び替えにおいてRownumber数式は使用できないので、これを削除します。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/29a38b50-f058-8e1f-f097-b973fc00ff59.png  
  
５．グループ用の列が表示されるのでグループを残して列のみを削除します。  
https://qiita-image-store.s3.amazonaws.com/0/47856/18c9f6d2-9550-74da-f312-92bc728b1231.png  
  
列を削除しようとすると「列の削除」ダイアログが表示されるので「列のみの削除」を選択します。  
https://qiita-image-store.s3.amazonaws.com/0/47856/7912c2eb-b0d6-0e21-af01-2e6055b23f25.png  
これにより、グループを残して列を削除できます。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/39c5bc33-aa76-89ed-58ea-dab8ac7e6010.png  
  
６．グループごとに改ページを行うようにします。  
グループ　プロパティーを開いたのち、改ページのタブで「グループの各インスタンスの間」にチェックを付与します。  
https://qiita-image-store.s3.amazonaws.com/0/47856/e111ee4a-3787-ac99-5d05-3202722d36d4.png  
  
７．これにより、１ページあたり20行で改行されるようになります。  
https://qiita-image-store.s3.amazonaws.com/0/47856/55ea75d0-b133-7821-6401-2a9ae12c5fcc.png  
  
### 一覧形式で最後のページにおいて最後まで罫線を引く  
特定の行数毎に改ページができることは説明しました。この方法でも、最後のページに大きな余白が作成されます。ここでは最後のページにおいても最後まで罫線を引く方法を説明します。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/b5452fbf-bd29-0aae-3560-43ae9750793c.png  
  
Excelの場合は、Excelオブジェクト経由で罫線の描画ができるので、プログラムでいかようにも調整できます。  
ReportViewerの場合はどうしたらいいのでしょうか？  
レイアウトで何かをするのは無理です。これはページがちょうどに終わるように、DataSetに格納するレコードの数を調整するしかありません。  
まず、DataSetのデザインで使用するすべてのフィールドのプロパティでAllowDbNullをTrueにします。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/6ac37943-8315-ea50-3ddd-a45f3934c089.png  
  
あとは、ReportViewを表示するまえに、ダミーのデータを追加します。  
  
```csharp
 private void Form1_Load(object sender, EventArgs e)
 {
 // TODO: このコード行はデータを 'DataSet1.DataTable1' テーブルに読み込みます。必要に応じて移動、または削除をしてください。
 this.DataTable1TableAdapter.Fill(this.DataSet1.DataTable1);

 // 末尾にダミーデータ追加
 int addcnt = 20 - (DataSet1.DataTable1.Count % 20);
 for (int i = 0; i < addcnt; ++i)
 {
 DataRow r = DataSet1.DataTable1.NewRow();
 DataSet1.DataTable1.Rows.Add(r);
 }

 this.reportViewer1.RefreshReport();
 }
```  
  
これで実行することで最後まで罫線が引かれていることがわかります。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/28f770d3-9e4d-e30a-a0a1-c997e3071984.png  
  
# ExcelとReportViewerの比較  
帳票の出力方法として、Excelオブジェクトを操作する方法があります。この章ではExcelを用いる方法とReportViewerを使用する方法で、どのような差があるか比較してみます。  
  
## ReportViewerのメリット  
ここではReportViewrを採用することで、Excelで帳票を作成する場合に比べてどのような利点が発生するかを説明します。  
  
１．ReportViewerではユーザ環境にOFFICE製品が不要になる。  
ReportViewerでは動作環境でOffice製品が不要になります。  
Excelを使用する場合、動作環境にExcelが存在しなければなりません。  
このことは動作する環境を選ぶことになります。特にパッケージソフトの場合、初めから、Office製品をもっていない層を販売対象から外すという選択が適切とは考えられません。  
なるべく多くの環境で動作させる必要がある場合、ReportViewerの採用は大きなメリットがあります。  
  
２．単票形式の帳票を作りやすい  
Excelは表計算ソフトなので、当然、単票形式の帳票を作るのに向いていません。  
シートをコピーするか、出力する際にレイアウトをプログラムで一々記述するか、１ページ出力するたびにExcelオブジェクトから作り直すか・・・いずれの方法も、開発の手間またはパフォーマンスに影響を与えます。  
  
それにくらべて、チュートリアルを読んでいただけると、ReportViewerでは単票形式のデータを作成しやすいことがわかります。  
  
## ReportViewerのデメリット  
ここでは逆にReportViewrにどのようなデメリットがあるか説明します。  
  
１．Excelとまっとく同じにはならない。  
ReportViewrはExcelにエクスポートする機能も有していますが、Excelとまったく同じものは作成できません。エンドユーザーがすでにExcelを駆使して業務をおこなっているシステムの場合、そのことが不満につながる場合があります。  
Excelで出力を提供できるなら、むりにそれを変更するのは実際使用するエンドユーザーのメリットにはならないでしょう。  
  
２．認知度の低さ  
ReportViewrはMicrosoftが提供しているコントロールですが、意外と使われていません。  
おそらく、検索しても大した情報を得ることはできないでしょう。  
いくら優れた機能を提供していても認知度が低いといくつかのリスクが発生します。  
A  
１つはトラブルシュートの事例の少なさです。Excelなどは大量にユーザーがいて様々な問題が報告され、それの対策がいくらでも取得できます。しかし、導入事例が少ないと、そのような情報を入手することが困難になります。  
事実、この資料を作成するときに、発生したいくつかのトラブルの解決方法は日本語ではヒットすらしません。  
  
次に開発要員の確保のしづらさです。Excelを触った人間はいくらでもいますが、ReportViewerを触ったことのある人間はそんなに多くありません。  
Excelで開発をおこなっている場合、いざとなったら、データを特定のシートにコピーするとこまでプログラマが開発して、実際のデザインなどはExcel使えるだけの人員にお願いするという開発体制がとれます。  
ReportViewerの場合、その方法はとれません。どんな簡単なデザインの修正であれ、プログラミングの知識がないと無理でしょう。  
  
  
# 付録  
## VS2008からSQLSERVER2012に接続する  
VisualStudio2008+SQLSERVER2012の場合、接続しようとしただけで下記のエラーが発生します。  
https://qiita-image-store.s3.amazonaws.com/0/47856/f4298a72-35f2-a86a-0f79-4e576f9cd826.png  
  
  
  
この場合は以下の点を確認してください。  
・VisualStudio2008SP1が適用されており、かつ、すべての重要な更新プログラムが当たっていること。  
・OLE DB用のデータプロバイダを使用する  
OLEDB用のデータプロバイダはデータソースの変更で選択できます。  
https://qiita-image-store.s3.amazonaws.com/0/47856/255c68e1-1acd-96eb-b22d-fd9f23246cc7.png  
  
## 帳票のテスト方法についての考察  
帳票のテストを行う場合、実際に紙に出力するのは、安定するまで待った方がいいでしょう。  
不安定な状況でのテストは紙と時間の無駄になります。  
  
幸いWindows Vista以降のOSにはXPS(XML Paper Specification)と呼ばれるドキュメントファイルがサポートされています。コントロールパネルのデバイスとプリンタを見ると、「Microsoft XPS Document Writer」と呼ばれるものがあります。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/f519da8f-9da1-7f5f-b86b-506b40c6e513.png  
  
通常のプリンタに出力するかわりにXPS Document Writerに出力することにより、紙ではなくファイルとして保存できます。  
  
動作が安定するまではこれを用いて動作確認をするといいでしょう。  
  
また、ファイルに保存できるということは機械的に以前出力した結果と変わっていないか確認することができます。これは回帰テストを自動で行えるということを意味します。  
  
印字のたびにファイル名を効かれるのを抑制するには、下記のプロパティに任意のファイル名を指定するといいでしょう。  
  
```csharp
System.Drawing.Printing.PrinterSettings
```  
  
ただし、単純にXPSを比較した場合、まったく同じ出力結果であっても、作成日や作成者によるメタデータによって違う結果とみなされます。これを防ぐにはXPSを一旦、画像に変換して比較を行うとよいでしょう。  
  
XpsToImg.zip  
http://needtec.sakura.ne.jp/release/XpsToImg.zip  
任意のXPSファイルをPNGファイルに変換するツールです。  
  
当然、数式などで出力日、作成日を表示すると異なる結果になるので、レポートパラメータなどを用いて、テスト用に固定の文字を指定できるようにします。  
  
一つ注意しなければならないのは、XPSで出力できるからといって紙で出力する試験をやらなくていいわけではありません。  
出力するプリンタにより、思わぬ出力結果になることがあります。すべての帳票の改ページを含むデータで必ず１度は動作確認すべきです。  
