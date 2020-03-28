# はじめに  
UiPath StudioXなるものが登場したそうです。  
https://forum.uipath.com/t/uipath-studiox/159600  
  
>従来のプロファイル (Studio) と ビジネスユーザー向けの簡潔なプロファイル (StudioX) に切り替えられます。ビジネスユーザー向けのプロファイルでは、ボタン数もぐっと減り、アクティビティ名もテクノロジーにあまり明るくない人にもわかりやすく表示されています。  
  
# 試してみる  
## StudioXのプロファイルの切り替え  
(1)[License and Profile]の[View or Change Profile]を実行  
![image.png](/image/80eecf32-b850-1859-3576-8d94c9743f57.png)  
  
(2)UiPath StudioX を選択する  
![image.png](/image/6c4fb5c8-17da-03aa-8fd3-5a309dd10ffd.png)  
  
(3)再起動を聞かれるので「はい」  
![image.png](/image/c1e87174-d24b-885c-9862-b865e21d3cb0.png)  
  
(4)UiPath StudioXが開く  
![image.png](/image/2d4785b9-0675-35b3-6097-18c7d5cfca4e.png)  
  
(5)ホーム→ツール→Excelアドインをインストールする。  
![image.png](/image/e26506c9-9483-cee9-24a6-af9c581bcadf.png)  
  
![image.png](/image/15f5edff-225d-883a-c781-cfd15a83dd84.png)  
※うまくいかなかったらUiPathを一度再起動してみてください。  
  
## 簡単な操作をやってみる。  
電卓のボタンを押して、表示内容をExcelに張り付けるサンプル  
  
(1)タスクオートメーションの作成を行う  
![image.png](/image/4f089579-fc91-11bd-3344-f0fbada65ed3.png)  
  
(2)以下のような画面が開く  
![image.png](/image/53553b7b-7fc6-a7f8-852d-5b5507db08e8.png)  
  
  
(3)アプリケーションカード等を設置する  
![image.png](/image/78aa44ae-2a40-ccaf-2871-e11c0c40178e.png)  
  
(4)アプリケーションカード内の「アプリケーションを指定」を指定すると画面上のWindowsを選択できるので電卓を選択する。  
![image.png](/image/51fb5dfc-9189-a09e-e94f-b0c1d00e2b17.png)  
  
(5)「クリック」アクティビティをアプリケーションカード内にドロップして「ターゲットの選択」後、電卓上のボタンをクリックする。  
![image.png](/image/97202989-6e45-7aad-69e3-c2fa0072ab3f.png)  
![image.png](/image/f596c90f-9807-ce03-abd4-5feda0ae5602.png)  
  
(6)テキストの取得をドロップ後、ターゲットの指定で電卓のテキストボックスをクリックする。  
![image.png](/image/aa479090-2429-302b-3e26-b08bff32145f.png)  
![image.png](/image/d6dd6dfd-1d32-2991-28ce-b5f4af42a438.png)  
  
(7)「テキストを保存」アクティビティの保存先で「Excel内で示す」を選択  
![image.png](/image/7aab91c5-ff3e-fcd8-c7d2-7806ee4b56b6.png)  
  
(8)Excelが起動するのでテキストを保存したいセルを選択して「Confirm」を押す  
![image.png](/image/dcb00817-3a70-c00e-0641-620fd9079b72.png)  
  
**実行結果**  
![uipath1.gif](/image/33611e61-11a6-7783-ae5f-6842281e2582.gif)  
  
**プロファイルを戻して開き直した場合**  
![image.png](/image/209b2385-3df7-08ba-c3f7-5133ace7e73d.png)  
※StudioXで作ったものを従来のもので開くことは可能ですが、逆は無理そうです。  
  
# 感想  
たしかにVB.NETの知識がなくても組めるようになったようなので「**テクノロジーにあまり明るくない人にもわかりやすく**」なったかと思います。  
