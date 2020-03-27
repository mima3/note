# はじめに  
Webアプリケーションに対してある種の繰り返しの操作を行ったり、定型処理を定期的に自動実行したい場合がよくあります。  
大きくわけてWebアプリケーションの自動化には３種類のやり方が存在します。  
  
1つ目は[ブラウザのGUI上の操作をプログラム上で真似して自動化する方法](#ブラウザのGUI上の操作をプログラム上で真似して自動化する方法)  
2つ目は[ブラウザから送信しているデータを真似する方法](#ブラウザから送信しているデータを真似する方法)  
3つ目は[Webアプリケーションが提供しているAPIを利用する方法](#Webアプリケーションが提供しているAPIを利用する方法)  
  
1つ目のブラウザのGUI上の操作をプログラム上で真似して自動化する方法は直観的にわかりやすいと言われますが、実際は最も難しい自動化の方法になります。また、アプリケーションのバージョンアップに伴い自動化用のプログラムが動作しなくなる可能性があります。  
  
2つ目のブラウザから送信しているデータを真似する方法はプログラムで実装しやすいやり方ではありますが、Webアプリケーションがどのようなデータを送信しているかなどを調べる必要があります。また、１つ目と同様にアプリケーションのバージョンアップに伴い自動化用のプログラムが動作しなくなる可能性があります。  
  
3つ目のWebアプリケーションが提供しているAPIを利用する方法が最も簡単でかつ正確に自動操作を行えます。ただし、WebアプリケーションのAPIが提供されているかどうかは自動操作対象のWebアプリケーションの仕様次第です。  
  
  
# 実験環境  
Windows10+PowerShell5.1  
Visual Studio 2019　+ .NET Framework 4.6  
Python 3.7.4  
NodeJs v10.16.0  
UiPath 2019.8.0-beta 83  
  
# ブラウザのGUI上の操作をプログラム上で真似して自動化する方法  
Windowsの場合、ブラウザを操作して自動化する方法も大きく４つの方法があります。  
  
１つ目の方法は[InternetExploreのCOMを利用する方法](#Internet ExploreのCOMを使用した自動化)です。WindowsのInternetExploreに対してならば、なにもインストールすることなく、ブラウザの自動操作が可能になっています。ただし、InternetExploreの寿命自体が、長く持たない可能性があるので注意が必要です。（InternetExplore自体が持っても、対応するWebアプリケーションがInternetExploreのサポートを切る可能性が高いです）  
  
２つ目の方法として[Seleniumを使用する方法](#Seleniumを使用した自動化)です。外部のライブラリが必要になりますが、多くのブラウザがサポートされています。  
  
３つ目の方法としてはブラウザが提供している[拡張機能を作成する方法](#拡張機能を作成する方法)です。ChromeとFirefoxの場合、JavaScriptで作成できるので、外部のライブラリを導入する必要はありません。  
  
４つ目の方法として[RPAツールを使用する方法](#RPAツールを使用する方法)です。ブラウザ上の要素について深く考えなくても、GUIベースで自動化が行えますが、コストの面で問題になります。  
  
## HTMLの要素を調べ方  
どのような方法でブラウザを操作するとしても、HTMLがどのような要素で構成されているかを調べる必要があります。  
ここではChromeでGoogleで検索する場合を例として画面上の要素を調べる方法を説明します。  
  
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/d8904265-20ce-119d-2dbf-130c6b9003c0.png  
  
１．ChromeにてF12キーを押下して開発者ツールを開きます。その後、「Elements」タブを選択してください。  
  
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/ab20fede-c93a-fffc-0afb-973ceac2b5e7.png  
  
２．[CTRL]+[Shift]+[C]を押下するか、下記のアイコンをクリックします。  
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/516d2b95-82c6-bd48-f9ec-e4bcd24a019d.png  
  
３．調べたい要素にマウスを移動させます。  
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/aa3e92e2-7139-b075-a7cc-5009dc18d1b4.png  
  
４．Elementsタブに選択した要素の内容が表示されます。今回の場合、以下のような内容が表示されます。  
  
```html
<input class="gLFyf gsfi" maxlength="2048" name="q" type="text" jsaction="paste:puy29d" aria-autocomplete="both" aria-haspopup="false" autocapitalize="off" autocomplete="off" autocorrect="off" role="combobox" spellcheck="false" title="検索" value="" aria-label="検索" data-ved="0ahUKEwiC0u6iu4nlAhXwyIsBHWwTBHcQ39UDCAQ">
```  
  
inputタグの属性が以下のようになっていることがわかります。  
  
|属性|値|  
|:---|:---|  
|class|gLFyf gsfi|  
|name|q|  
  
自動操作を行う場合、id、name、classなどを利用して要素を指定することになるので、属性値をメモしておきましょう。  
  
５．同様にボタンについても属性を調べます。その結果は以下のようになります。  
  
```html
<input class="gNO89b" value="Google 検索" aria-label="Google 検索" name="btnK" type="submit" data-ved="0ahUKEwiC0u6iu4nlAhXwyIsBHWwTBHcQ4dUDCAo">
```  
  
|属性|値|  
|:---|:---|  
|class|gNO89b|  
|name|btnK|  
  
ここで調べた属性を利用して要素を特定して自動操作を行うことになります。。  
また、今回はChromeでのやり方を紹介しましたが、他のブラウザでも同様のことが可能です。同じWebアプリケーションを使用していてもブラウザによって出力される内容が異なる可能性もあるので、自動操作を行うブラウザを使用して要素を調べるようにしましょう。  
  
### InternetExplore11の場合  
IE11でもF12キーを押すことで開発者ツールが表示されます。  
そこで「DOM Explore」タブを開いて要素の選択を行うことで要素の属性を調べることが可能です。  
  
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/2c011407-27b8-c591-e3a3-b3563d99c7f7.png  
  
### Edgeの場合  
Edgeは将来Chromeベースのものに置き換わる可能性があります。  
今回は旧EdgeとChromeベースの新Edge両方について説明します。  
  
#### 旧Edge  
F12キーで開発者ツールが表示されます。  
そこで「要素」タブを開いて要素の選択を行うことで要素の属性を調べることが可能です。  
  
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/ac850ea1-7e24-c5ae-072d-89b072974018.png  
  
#### 新Edge  
2019年10月時点でベータ版としてリリースされているEdgeの場合、F12キーで開発者ツールが表示されます。  
Chromeと同様の操作で要素の属性を調べることが可能です。  
  
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/42036d9b-373c-2644-edf5-3455fa839320.png  
  
### Firefoxの場合  
F12キーで開発者ツールが表示されます。  
そこで「インスペクター」タブを開いて要素の選択を行うことで要素の属性を調べることが可能です。  
  
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/79741a0b-3376-360d-5760-fd45718d063f.png  
  
### HTMLの要素を調べる方のまとめ  
多くのブラウザは開発者ツールをサポートしており、要素の属性を調べることが可能です。  
Webアプリケーションの自動化では、この要素を特定して操作する必要があるので、お使いのブラウザでの要素の調べ方は覚えておきましょう。  
  
実際の自動操作の例は次章から解説します。  
  
## Internet ExploreのCOMを使用した自動化  
Internet ExploreのCOMである[mshtml](https://docs.microsoft.com/ja-jp/previous-versions/windows/internet-explorer/ie-developer/platform-apis/hh801968(v=vs.85))を経由することでInternetExploreを自動操作できます。  
  
>COM(Component Object Model)  
Microsoftが提唱した再利用を目的とした技術で、COMを用いて開発した部品はプログラム言語に依存せずに利用できるようになります。たとえば、説明したInternetExploreの操作やOfficeアプリケーションなどが外部から利用できるのは、COMのおかげです。  
  
  
### IE操作の単純な例  
実際のGoogleのトップページで任意の単語を検索するサンプルを見てみましょう。  
  
#### WSHやVBAでのIE操作の単純な例  
WSHのVBScritpで以下のように実装可能です。  
以下コードはWScript.EchoをDebug.Print等に置き換えることでVBAでも流用できると思います。  
  
```vb
Dim ie
Set ie = CreateObject("InternetExplorer.Application")
ie.Visible = True 
call ie.Navigate("https://www.google.com/")

'ページが読み込まれるまで待機
Do While ie.Busy = True Or ie.readyState <> 4
    WScript.Sleep 100        
Loop

Dim doc
Set doc = ie.Document
Dim txt
Set txt = doc.getElementsByName("q")
txt.item(0).value = "ドリフターズ"

Dim btn
Set btn = doc.getElementsByName("btnK")
btn.item(0).click()

'ページが読み込まれるまで待機
Do While ie.Busy = True Or ie.readyState <> 4
    WScript.Sleep 100        
Loop
Set doc = ie.Document

Dim list
Do While True
    Set list = doc.getElementsByClassName("LC20lb")

    If Not list is Nothing Then
        If list.length > 0 Then
            Exit Do
        End If
    End If
    WScript.Sleep 100 
Loop

Dim item
For Each item In list
    WScript.Echo item.innerText
Next
ie.Quit
```  
  
このサンプルでは[Navigate](https://docs.microsoft.com/ja-jp/previous-versions/windows/internet-explorer/ie-developer/platform-apis/aa752093(v=vs.85))で指定のURLに移動したのち、[getElementsByName](https://docs.microsoft.com/ja-jp/previous-versions/windows/internet-explorer/ie-developer/platform-apis/aa752544(v=vs.85))を使用して属性を取得して、値の設定とクリック操作を行っています。  
その後,[Busy](https://docs.microsoft.com/ja-jp/previous-versions/windows/internet-explorer/ie-developer/platform-apis/aa752050(v=vs.85))と[readyState](https://docs.microsoft.com/ja-jp/previous-versions/windows/internet-explorer/ie-developer/platform-apis/jj160966(v=vs.85))を監視してページの切り替え完了を待ちます。  
その後、検索結果の要素が出現するまで待機して、その要素の内容を出力します。  
  
##### 「起動されたオブジェクトはクライアントから切断されました。」エラーが出る場合  
多くの場合、上記の方法で自動操作ができますが同じページを同じソースコードで実行してもエラーが発生する場合と発生しない場合があります。  
  
起動されたオブジェクトはクライアントから切断されました。  
```

このエラーがでた場合は表示対象ページのプロパティを確認してみてください。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/f9d65178-f0f6-54bb-32dd-d538eb9fff02.png

https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/49f10d41-c7f1-8e6f-524d-bf214b3bb61a.png

保護モードが無効になっているかと思います。

１つのIEのプロセスには整合性レベル（IL）が1つしかないため、インターネット（保護モード、LowIL）とイントラネット（非保護モード、MediumIL）を同時に動かすことはできず、、Internet Explorerはデフォルトのタブプロセスが[低整合性]タブであると想定しているため、保護モードがオフのサイトに移動した時点で制御を失います。

これに対応するには非保護モードのIEを起動する必要があります。

```vb  
Dim ie  
'CLSIDを直接使用して非保護モードのIEを起動  
Set ie = GetObject("new:{D5E8041D-920F-45e9-B8FB-B1DEB82C6E5E}")  
  
ie.Visible = True   
call ie.Navigate("https://www.google.com/")  
  
  
'ページが読み込まれるまで待機  
Do While ie.Busy = True Or ie.readyState <> 4  
    WScript.Sleep 100        
Loop  
```

**Default Integrity Level and Automation**
https://blogs.msdn.microsoft.com/ieinternals/2011/08/03/default-integrity-level-and-automation/

#### PowerShellでのIE操作の単純な例
実はInternetExploreのCOM操作についてPowerShellやC#などの.NETでの実装は厄介です。結論から言えばやめといた方がいいです。
たとえば、よく見かける実装でGoogleとはてなブックマークを検索するスクリプトieng1.ps1とieng2.ps1を用意しました。

**Googleの検索**

```powershell:ieng1.ps1  
$ie = New-Object -ComObject InternetExplorer.Application  # IE起動  
  
$ie.Visible = $true  
$ie.Navigate("https://www.google.com/")  
  
  
# ページが読み込まれるまで待機  
while ($ie.Busy -or $ie.readyState -ne 4) {  
    Start-Sleep -Milliseconds 100  
}  
$doc=$ie.Document  
  
# 検索文字入力  
$txt=$doc.getElementsByName("q")  
$txt[0].value = "ドリフターズ"  
  
# ボタン押下  
$btn=$doc.getElementsByName("btnK")  
$btn[0].click()  
  
# ページが読み込まれるまで待機  
while ($ie.Busy -or $ie.readyState -ne 4) {  
    Start-Sleep -Milliseconds 100  
}  
$doc = $ie.Document  
  
# LC20lbが表示されるまで待機  
while($true) {  
  $list = $doc.getElementsByClassName('LC20lb')  
  if($list -and $list.length -ge 1) { break }  
  Start-Sleep -Milliseconds 100  
}  
  
foreach($i in $list){  
  Write-Host($i.innerText)  
}  
$ie.Quit()  
Write-Host("OK")  
```

**はてなの検索**

```powershell:ieng2.ps1  
$ie = New-Object -ComObject InternetExplorer.Application  # IE起動  
  
$ie.Visible = $true  
$ie.Navigate("https://b.hatena.ne.jp/")  
  
  
# ページが読み込まれるまで待機  
while ($ie.Busy -or $ie.readyState -ne 4) {  
    Start-Sleep -Milliseconds 100  
}  
$doc=$ie.Document  
  
# 検索文字入力  
$txt=$doc.getElementsByName("q")  
$txt[0].value = "ドリフターズ"  
  
# ボタン押下  
$btn=$doc.getElementsByClassName("gh-search-button")  
$btn[0].click()  
  
# ページが読み込まれるまで待機  
while ($ie.Busy -or $ie.readyState -ne 4) {  
    Start-Sleep -Milliseconds 100  
}  
$doc = $ie.Document  
  
# LC20lbが表示されるまで待機  
while($true) {  
  $list = $doc.getElementsByClassName('centerarticle-entry-title')  
  if($list -and $list.length -ge 1) { break }  
  Start-Sleep -Milliseconds 100  
}  
  
foreach($i in $list){  
  Write-Host($i.innerText)  
}  
$ie.Quit()  
Write-Host("OK")  
```

この実装はいくつか問題を起こす可能性があります。

##### PowerShellでのIE操作の問題点
###### 環境によっては動作しない
同じコードをPowerShell2.0+Windows7で動かそうとしたところ下記のエラーを表示して動作しません。

```  
PS C:\share\webctrl> $ie.Document.getElementsByName("q")  
"getElementsByName" のオーバーロードで、引数の数が "1" であるものが見つかりません。  
発生場所 行:1 文字:31  
+ $ie.Document.getElementsByName <<<< ("q")  
    + CategoryInfo          : NotSpecified: (:) []、MethodException  
    + FullyQualifiedErrorId : MethodCountCouldNotFin  
```

この問題は下記のフォーラムで議論されていますが、解決はしていません。

 - [powershellのIEを操作するスクリプトの中のgetElementByIdを使う箇所でエラーが起こる](https://social.technet.microsoft.com/Forums/ja-JP/02cd48a5-5236-4a4e-8a66-0c8a5b67ceaf/powershell12398ie12434258052031612377124271247312463125221250312488?forum=powershellja)

###### COMの解放処理を行っていない
上記のコードはCOMを.NETから利用しているにも関わらずReleaseComObjectを行っていません。
類似の問題として以下を参照してください。

**.NETを使ったOfficeの自動化が面倒なはずがない―そう考えていた時期が俺にもありました。**
https://qiita.com/mima_ita/items/aa811423d8c4410eca71

解放処理を行ったコードは以下のようになります。

```powershell  
	$ie = New-Object -ComObject InternetExplorer.Application  # IE起動  
  
	$ie.Visible = $true  
	$ie.Navigate("https://www.google.com/")  
  
  
	# ページが読み込まれるまで待機  
	while ($ie.Busy -or $ie.readyState -ne 4) {  
	  Start-Sleep -Milliseconds 100  
	}  
	$doc=$ie.Document  
  
	# 検索文字入力  
	$txt=$doc.getElementsByName("q")  
	$txt[0].value = "ドリフターズ"  
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($txt) | Out-Null  
	Remove-Variable txt -ErrorAction SilentlyContinue  
  
	# ボタン押下  
	$btn=$doc.getElementsByName("btnK")  
	$btn[0].click()  
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($btn) | Out-Null  
	Remove-Variable btn -ErrorAction SilentlyContinue  
  
	## ページが読み込まれるまで待機  
	while ($ie.Busy -or $ie.readyState -ne 4) {  
	    Start-Sleep -Milliseconds 100  
	}  
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($doc) | Out-Null  
	$doc = $ie.Document  
  
	# LC20lbが表示されるまで待機  
	while($true) {  
	  $list = $doc.getElementsByClassName('LC20lb')  
	  if($list -and $list.length -ge 1) {   
	    break   
	  }  
	  if ($list) {  
	    [System.Runtime.Interopservices.Marshal]::ReleaseComObject($list) | Out-Null  
	  }  
	  Start-Sleep -Milliseconds 100  
	}  
  
	for($i=0; $i -lt $list.length; $i++) {  
	  $item = $list[$i]  
	  Write-Host($item.innerText)  
	  [System.Runtime.Interopservices.Marshal]::ReleaseComObject($item) | Out-Null  
	  Remove-Variable item -ErrorAction SilentlyContinue  
	}  
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($list) | Out-Null  
	Remove-Variable list -ErrorAction SilentlyContinue  
  
	$ie.Quit()  
	Write-Host("OK")  
  
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($doc) | Out-Null  
	Remove-Variable doc -ErrorAction SilentlyContinue  
  
  
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($ie) | Out-Null  
	Remove-Variable ie -ErrorAction SilentlyContinue  
  
	[System.GC]::Collect()  
	[System.GC]::WaitForPendingFinalizers()  
	[System.GC]::Collect()  
  
```


###### 同一プロセスで複数起動した場合にエラーになる
先にあげた２つのスクリプトはPowerShellを再起動した直後には、それぞれ動作しますが以下のように続けて実行するとエラーになります。

```  
正常に動作する  
>powershell ./ieng1.ps1  
>powershell ./ieng2.ps1  
  
2つ目のスクリプトの実行でエラーになる  
>./ieng1.ps1  
>./ieng2.ps1  
```

エラーの内容は下記の通りです。

```  
HRESULT からの例外:0x800A01B6  
発生場所 C:\dev\ps\webctrl\ieng2.ps1:14 文字:1  
+ $txt=$doc.getElementsByName("q")  
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~  
    + CategoryInfo          : OperationStopped: (:) [], NotSupportedException  
    + FullyQualifiedErrorId : System.NotSupportedException  
```

この問題は以下の記事で言及されています。

 - [PowerShell Exception 0x800A01B6 while using getElementsByTagName, getElementsByName or getElementByID](https://paullimblog.wordpress.com/2017/07/28/powershell-exception-0x800a01b6-while-using-getelementsbytagname-getelementsbyname-or-getelementbyid/)
 - [Exception from HRESULT: 0x800A138A At line......](https://social.technet.microsoft.com/Forums/ie/en-US/33be53ea-3afe-49e7-b586-41de95593373/exception-from-hresult-0x800a138a-at-line?forum=winserverpowershell)

getElementsByNameをIHTMLDocument3_getElementsByNameに置き換えて、getElementsByClassNameをInvokeMemberで呼び出すようにすれば回避できるようです。

**Googleの検索（修正版)**

```powershell:ieok1.ps1  
	$ie = New-Object -ComObject InternetExplorer.Application  # IE起動  
  
	$ie.Visible = $true  
	$ie.Navigate("https://www.google.com/")  
  
  
	# ページが読み込まれるまで待機  
	while ($ie.Busy -or $ie.readyState -ne 4) {  
	  Start-Sleep -Milliseconds 100  
	}  
	$doc=$ie.Document  
  
	# 検索文字入力  
	$txt=$doc.IHTMLDocument3_getElementsByName("q")  
	$txt[0].value = "ドリフターズ"  
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($txt) | Out-Null  
	Remove-Variable txt -ErrorAction SilentlyContinue  
  
	# ボタン押下  
	$btn=$doc.IHTMLDocument3_getElementsByName("btnK")  
	$btn[0].click()  
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($btn) | Out-Null  
	Remove-Variable btn -ErrorAction SilentlyContinue  
  
	## ページが読み込まれるまで待機  
	while ($ie.Busy -or $ie.readyState -ne 4) {  
	    Start-Sleep -Milliseconds 100  
	}  
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($doc) | Out-Null  
	$doc = $ie.Document  
  
	# LC20lbが表示されるまで待機  
	while($true) {  
	  $list = [System.__ComObject].InvokeMember("getElementsByClassName", [System.Reflection.BindingFlags]::InvokeMethod, $null, $doc, "LC20lb")  
  
	  if($list -and $list.length -ge 1) {   
	    break   
	  }  
	  if ($list) {  
	    [System.Runtime.Interopservices.Marshal]::ReleaseComObject($list) | Out-Null  
	  }  
	  Start-Sleep -Milliseconds 100  
	}  
  
	for($i=0; $i -lt $list.length; $i++) {  
	  $item = $list[$i]  
	  Write-Host($item.innerText)  
	  [System.Runtime.Interopservices.Marshal]::ReleaseComObject($item) | Out-Null  
	  Remove-Variable item -ErrorAction SilentlyContinue  
	}  
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($list) | Out-Null  
	Remove-Variable list -ErrorAction SilentlyContinue  
  
	$ie.Quit()  
	Write-Host("OK")  
  
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($doc) | Out-Null  
	Remove-Variable doc -ErrorAction SilentlyContinue  
  
  
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($ie) | Out-Null  
	Remove-Variable ie -ErrorAction SilentlyContinue  
  
	[System.GC]::Collect()  
	[System.GC]::WaitForPendingFinalizers()  
	[System.GC]::Collect()  
  
```

**はてなの検索（修正版)**

```powershell:ieok2.ps1  
	$ie = New-Object -ComObject InternetExplorer.Application  # IE起動  
  
	$ie.Visible = $true  
	$ie.Navigate("https://b.hatena.ne.jp/")  
  
  
	# ページが読み込まれるまで待機  
	while ($ie.Busy -or $ie.readyState -ne 4) {  
	  Start-Sleep -Milliseconds 100  
	}  
	$doc=$ie.Document  
  
	# 検索文字入力  
	$txt=$doc.IHTMLDocument3_getElementsByName("q")  
	$txt[0].value = "ドリフターズ"  
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($txt) | Out-Null  
	Remove-Variable txt -ErrorAction SilentlyContinue  
  
	# ボタン押下  
	$btn=[System.__ComObject].InvokeMember("getElementsByClassName", [System.Reflection.BindingFlags]::InvokeMethod, $null, $doc, "gh-search-button")  
	$btn[0].click()  
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($btn) | Out-Null  
	Remove-Variable btn -ErrorAction SilentlyContinue  
  
	## ページが読み込まれるまで待機  
	while ($ie.Busy -or $ie.readyState -ne 4) {  
	    Start-Sleep -Milliseconds 100  
	}  
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($doc) | Out-Null  
	$doc = $ie.Document  
  
	#表示されるまで待機  
	while($true) {  
	  $list = [System.__ComObject].InvokeMember("getElementsByClassName", [System.Reflection.BindingFlags]::InvokeMethod, $null, $doc, "centerarticle-entry-title")  
  
	  if($list -and $list.length -ge 1) {   
	    break   
	  }  
	  if ($list) {  
	    [System.Runtime.Interopservices.Marshal]::ReleaseComObject($list) | Out-Null  
	  }  
	  Start-Sleep -Milliseconds 100  
	}  
  
	for($i=0; $i -lt $list.length; $i++) {  
	  $item = $list[$i]  
	  Write-Host($item.innerText)  
	  [System.Runtime.Interopservices.Marshal]::ReleaseComObject($item) | Out-Null  
	  Remove-Variable item -ErrorAction SilentlyContinue  
	}  
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($list) | Out-Null  
	Remove-Variable list -ErrorAction SilentlyContinue  
  
	$ie.Quit()  
	Write-Host("OK")  
  
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($doc) | Out-Null  
	Remove-Variable doc -ErrorAction SilentlyContinue  
  
  
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($ie) | Out-Null  
	Remove-Variable ie -ErrorAction SilentlyContinue  
  
	[System.GC]::Collect()  
	[System.GC]::WaitForPendingFinalizers()  
	[System.GC]::Collect()  
  
```

###### 非保護モードのIEをPowerShellで操作する場合
VBSと同様に非保護モードのIEはこのままでは操作できません。
非保護モードのIEを以下のようにして起動する必要があります。

```powershell  
	# 非保護モードのIE起動  
	$clsid = New-Object Guid "D5E8041D-920F-45e9-B8FB-B1DEB82C6E5E"  
	$type = [Type]::GetTypeFromCLSID($clsid)  
	$ie = [Activator]::CreateInstance($type)  
  
	$ie.Visible = $true  
	$ie.Navigate("https://www.google.com/")  
  
  
	# ページが読み込まれるまで待機  
	while ($ie.Busy -or $ie.readyState -ne 4) {  
	  Start-Sleep -Milliseconds 100  
	}  
```

**Accessing COM Objects without ProgID**
https://community.idera.com/database-tools/powershell/powertips/b/tips/posts/accessing-com-objects-without-progid

あるいは非保護モードのIEのオブジェクトを取り直す手法もあります。

```powershell  
	$ie = New-Object -ComObject InternetExplorer.Application  # IE起動  
  
	$ie.Visible = $true  
	$ie.Navigate("https://www.google.com/")  
  
	# 非保護モード対応  
	$hwnd = $ie.HWND  
	[System.Reflection.Assembly]::LoadFrom("C:\Program Files (x86)\Microsoft.NET\Primary Interop Assemblies\Microsoft.mshtml.dll")  
	$shell = New-Object -ComObject Shell.Application  
	while ($ie.Document -isnot [mshtml.HTMLDocumentClass]) {  
		$ie = $shell.Windows() | ? {$_.HWND -eq $hwnd}  
	}  
  
	# ページが読み込まれるまで待機  
	while ($ie.Busy -or $ie.readyState -ne 4) {  
	  Start-Sleep -Milliseconds 100  
	}  
```

**PowershellでInternetExplorerを操作する**
https://qiita.com/flasksrw/items/a1ff5fbbc3b660e01d96

###### PowerShellでのInternetExploreの操作のまとめ
**IEのCOM操作は辞めといた方がいいでしょう。**

回避策を書いているページは色々見つかりますが、何故その事象が発生しているか、そしてなぜ回避することができたかを説明できているサイトは見つからず、**ためしてみたら動作した**以上のものではありません。
固定の環境で動かすスクリプトならばともかく、不特定多数の環境で動作するスクリプトでは採用を避けるべきでしょう。また、多くの場合、VBSやVBAで代替できるのでPowerShellにこだわる理由はないでしょう。

どうしても、PowerShellでIEのCOM操作を行いたい場合は、IE操作をしたプロセスを終了するようにすると安定しそうです。

**なお、PowerShellでWebのGUIの自動操作を行いたい場合は、後述のSeleniumを利用した方が安定すると思います。**


### すでに起動しているブラウザの操作方法
すでに起動しているIEを操作するにはIEのウィンドウに対して[RegisterWindowMessage](https://docs.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-registerwindowmessagew)でWM_HTML_GETOBJECTメッセージを定義して送信することでHTMLDocumentを取得することになります。

**Cant create HTML document from Hwnd using C#**
https://stackoverflow.com/questions/20873885/cant-create-html-document-from-hwnd-using-c-sharp

Win32 APIを直接実行できないWSHでは実現が難しいです。

#### VBAでの実装例
下記を参考にしてください。

**VBAでInternetExplore上のJavaScriptを無理やり動かすよ！**
https://qiita.com/mima_ita/items/fdff129a8db1153c9940

#### C#での実装例
.NET経由になるのでCOMの解放処理を入れる必要があります。

(1)参照マネージャーのCOMタブでMicrosoft HTML Object Libraryを追加します。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/39525911-75d6-2dcb-7e13-183638402e77.png

これにより「Interop.MSHTML.dll」が作成されます。
Interop～.dllは[tlbImpコマンド](https://docs.microsoft.com/en-us/dotnet/framework/interop/how-to-generate-interop-assemblies-from-type-libraries)を使用することでコマンドラインで作成できますが、VisualStudioなどの開発ツールをインストールしていないと使えないと思います。

(2)以下のような実装をします。下記のコードは.NET2.0でも動作します。

```csharp  
using System;  
using System.Collections.Generic;  
using System.Text;  
using System.Runtime.InteropServices;  
using MSHTML;  
using System.Diagnostics;  
  
namespace iesample  
{  
    class Program  
    {  
  
  
        [DllImport("user32.dll", EntryPoint = "GetClassNameA")]  
        public static extern int GetClassName(IntPtr hwnd, StringBuilder lpClassName, int nMaxCount);  
  
        /*delegate to handle EnumChildWindows*/  
        public delegate int EnumProc(IntPtr hWnd, ref IntPtr lParam);  
  
  
        [DllImport("user32.dll")]  
        public static extern int EnumWindows(EnumProc lpEnumFunc, ref IntPtr lParam);  
  
  
        [DllImport("user32.dll")]  
        public static extern int EnumChildWindows(IntPtr hWndParent, EnumProc lpEnumFunc, ref IntPtr lParam);  
  
        [DllImport("user32.dll", EntryPoint = "RegisterWindowMessageA")]  
        public static extern int RegisterWindowMessage(string lpString);  
  
        [DllImport("user32.dll", EntryPoint = "SendMessageTimeoutA")]  
        public static extern int SendMessageTimeout(IntPtr hwnd, int msg, int wParam, int lParam, int fuFlags, int uTimeout, out UIntPtr lpdwResult);  
  
        [DllImport("OLEACC.dll")]  
        public static extern int ObjectFromLresult(UIntPtr lResult, ref Guid _riid, int wParam, ref MSHTML.IHTMLDocument2 _ppvObject);  
  
        public const int SMTO_ABORTIFHUNG = 0x2;  
        public Guid IID_IHTMLDocument = new Guid("626FC520-A41E-11CF-A731-00A0C9082637");  
  
  
        private static List<IHTMLDocument2> cacheList = new List<IHTMLDocument2>();  
  
        public static MSHTML.IHTMLDocument2 FindBrowser(string title)  
        {  
            MSHTML.IHTMLDocument2 ret;  
            if (cacheList.Count == 0)  
            {  
                RefreshBrowserCache();  
            }  
            ret = FindBrowserInCache(title);  
            if (ret != null)  
            {  
                return ret;  
            }  
            RefreshBrowserCache();  
            return FindBrowserInCache(title);  
        }  
        public static MSHTML.IHTMLDocument2 FindBrowserInCache(string title)  
        {  
            MSHTML.IHTMLDocument2 ret;  
            foreach(MSHTML.IHTMLDocument2 item in cacheList)  
            {  
                if (item.title.Contains(title))  
                {  
                    return item;  
                }  
            }  
            return null;  
        }  
  
        public static void RefreshBrowserCache()  
        {  
            foreach (IHTMLDocument2 item in cacheList)  
            {  
                Marshal.ReleaseComObject(item);  
            }  
            cacheList.Clear();  
            EnumProc proc = new EnumProc(EnumIEWndProc);  
            IntPtr lparam = IntPtr.Zero;  
            EnumWindows(proc, ref lparam);  
        }  
  
        private static int EnumIEWndProc(IntPtr hWnd, ref IntPtr lParam)  
        {  
            StringBuilder className = new StringBuilder(128);  
            GetClassName(hWnd, className, className.Capacity);  
            EnumProc proc = new EnumProc(EnumIEServerWndProc);  
            if (className.ToString().Equals("IEFrame") || className.ToString().Equals("TabWindowClass"))  
            {  
                IntPtr lparam = IntPtr.Zero;  
                EnumChildWindows(hWnd, proc, ref lparam);  
            }  
            return 1;  
        }  
        private static int EnumIEServerWndProc(IntPtr hWnd, ref IntPtr lParam)  
        {  
            StringBuilder className = new StringBuilder(128);  
            GetClassName(hWnd, className, className.Capacity);  
            if (className.ToString().Equals("Internet Explorer_Server"))  
            {  
                IHTMLDocument2 doc = GetHTMLDocument(hWnd);  
                if (doc != null)  
                {  
                    cacheList.Add(doc);  
                }  
            }  
  
            return 1;  
        }  
        private static IHTMLDocument2 GetHTMLDocument(IntPtr hWnd)  
        {  
            int nMsg = RegisterWindowMessage("WM_HTML_GETOBJECT");  
            if (nMsg == 0)  
            {  
                return null;  
            }  
            UIntPtr lRes;  
            SendMessageTimeout(hWnd, nMsg, 0, 0, SMTO_ABORTIFHUNG, 1000, out lRes);  
            if (lRes == UIntPtr.Zero)  
            {  
                return null;  
            }  
  
            Guid IID_IHTMLDocument = new Guid("626FC520-A41E-11CF-A731-00A0C9082637");  
            IHTMLDocument2 doc = null;  
            int hr = ObjectFromLresult(lRes, ref IID_IHTMLDocument, 0, ref doc);  
            return doc;  
         }  
  
        static void Main(string[] args)  
        {  
            IHTMLDocument3 doc = FindBrowser("Google") as IHTMLDocument3;  
            IHTMLInputElement txt = doc.getElementsByName("q").item(0) as IHTMLInputElement;  
            txt.value = "ドリフターズ";  
  
            IHTMLElement btn = doc.getElementsByName("btnK").item(0) as IHTMLElement;  
            btn.click();  
  
  
  
            Marshal.ReleaseComObject(btn);  
            Marshal.ReleaseComObject(txt);  
            Marshal.ReleaseComObject(doc);  
  
        }  
    }  
}  
```

### 様々なコントロールを含むサンプルの例
様々なコントロールを含む以下のページの入力の自動化について考えてみます。
操作対象として以下のページを使用します。
http://needtec.sakura.ne.jp/auto_demo/form1.html

このページは「登録する」ボタンを押下することで確認メッセージが表示されて、「OK」の場合に登録処理を行うものとします。※
※実際はSleepしているだけでなにもしていないです。

WSHやVBSでの操作例は以下のようになります。

```vb  
Dim ie  
Set ie = CreateObject("InternetExplorer.Application")  
ie.Visible = True   
call ie.Navigate("http://needtec.sakura.ne.jp/auto_demo/form1.html")  
  
'ページが読み込まれるまで待機  
Do While ie.Busy = True Or ie.readyState <> 4  
    WScript.Sleep 100  
Loop  
  
Dim doc  
Set doc = ie.Document  
' INPUTBOX  
doc.getElementsByName("name").item(0).value = "名前太郎"  
doc.getElementsByName("mail").item(0).value = "test@co.jp"  
  
' テキストエリア  
doc.getElementsByName("comment").item(0).innerText = "猫猫子猫" & vbCrLf & "犬犬子犬"  
  
' チェックボックス  
doc.getElementsByName("q1[]").item(0).Checked = True  
doc.getElementsByName("q1[]").item(1).Checked = True  
  
' ラジオボタン  
doc.getElementsByName("men").item(1).Checked = True  
  
' 複数選択リスト  
Dim objSelect  
Set objSelect = doc.getElementsByName("osi[]").item(0)  
objSelect.getElementsByTagName("option").item(1).Selected = True  
objSelect.getElementsByTagName("option").item(2).Selected = True  
  
' ボタン押下  
' 確認メッセージ処理を偽造する  
doc.parentWindow.ExecScript("confirm = function () { return true; }")  
doc.getElementsByTagName("input").item(8).click()  
```

確認ダイアログを突破する方法は色々ありますが、上記でやったようなJavaScirptのconfirmやalertを上書きしてしまうのが最も楽だと思います。

#### 確認メッセージの処理を上書きしたくない場合
UIAutomation等を使用します。
確認メッセージが表示されるまで待機してボタンを押下するような処理を別スレッドかプロセスで起動します。WSHやVBAの場合は別プロセスでやった方が楽です。

まず、確認メッセージを監視するようなスクリプトを記載します。
これはPowerShellで書いた方が楽だと思います。

```powershell:wait_confirm.ps1  
Add-Type -AssemblyName UIAutomationClient  
Add-Type -AssemblyName UIAutomationTypes  
Add-type -AssemblyName System.Windows.Forms  
  
$source = @"  
using System;  
using System.Windows.Automation;  
public class AutomationHelper  
{  
    public static AutomationElement RootElement  
    {  
        get  
        {  
            return AutomationElement.RootElement;  
        }  
    }  
  
    public static AutomationElement GetWindowByTitle(string title) {  
        var rootCond = new PropertyCondition(AutomationElement.ClassNameProperty, "Alternate Modal Top Most");  
        var cond = new PropertyCondition(AutomationElement.NameProperty, title);  
        var elementCollection = RootElement.FindAll(TreeScope.Children, rootCond);  
        foreach(AutomationElement mainForm in elementCollection) {  
            var win =  mainForm.FindFirst(TreeScope.Children, cond);  
            if (win != null) {  
                return win;  
            }  
        }  
        return null;  
    }  
  
    public static AutomationElement WaitWindowByTitle(string title, int timeout = 10) {  
        DateTime start = DateTime.Now;  
        while (true) {  
            AutomationElement ret = GetWindowByTitle(title);  
            if (ret != null) {  
                return ret;  
            }  
            TimeSpan ts = DateTime.Now - start;  
            if (ts.TotalSeconds > timeout) {  
               return null;  
            }  
            System.Threading.Thread.Sleep(100);  
        }  
    }  
  
}  
"@  
Add-Type -TypeDefinition $source -ReferencedAssemblies("UIAutomationClient", "UIAutomationTypes")  
  
# 5.0以降ならusingで記載した方が楽。  
$autoElem = [System.Windows.Automation.AutomationElement]  
  
# ウィンドウ以下で指定の条件に当てはまるコントロールを１つ取得  
function findFirstElement($form, $condProp, $condValue) {  
    $cond = New-Object -TypeName System.Windows.Automation.PropertyCondition($condProp, $condValue)  
    return $form.FindFirst([System.Windows.Automation.TreeScope]::Element -bor [System.Windows.Automation.TreeScope]::Descendants, $cond)  
}  
  
# 指定のAutomationIDのボタンを押下  
function pushButtonById($form, $id) {  
    $buttonElem = findFirstElement $form $autoElem::AutomationIdProperty $id  
    $invElm = $buttonElem.GetCurrentPattern([System.Windows.Automation.InvokePattern]::Pattern) -as [System.Windows.Automation.InvokePattern]  
    $invElm.Invoke()  
}  
  
  
#  
$dialog = [AutomationHelper]::WaitWindowByTitle("Web ページからのメッセージ", 30)  
if ($dialog -eq $null) {  
    Write-Host "タイムアウト"  
} else {  
    pushButtonById $dialog "1"  
    Write-Host "終了"  
}  
```

あとはWSH側のボタン押下処理を以下のように修正します。

```vb  
' ボタン押下  
' 確認メッセージ処理を別プロセスで行う  
Dim shell  
Set shell = CreateObject("WScript.Shell")  
shell.Run "C:\Windows\System32\WindowsPowerShell\v1.0\powershell -ExecutionPolicy RemoteSigned -File C:\dev\ps\webctrl\wait_confirm.ps1", 0, False  
  
doc.getElementsByTagName("input").item(8).click()  
  
```

### Internet ExploreのCOMを使用した自動化のまとめ
VBAやVBSなどの何処でもはいっていそうなプログラミング言語で自動化できるのは強みです。
PowerShellでも行えますが、.NET経由だとCOMの解放処理が面倒だったり、動作が安定しない環境もあるので環境を制御できる場合のみに使用したほうがいいでしょう。

また、InternetExploreのサポートがいつまで続くかわからない以上、外部のツールを導入可能である場合は、お勧めしません。

## Seleniumを使用した自動化
様々なOS上の様々なブラウザを様々なプログラミング言語で自動操作するためのツールです。
Webアプリケーションをブラウザ操作で自動化する場合、もっともよく使われるツールになります。
[Selenium実践入門](https://www.amazon.co.jp/dp/B07JHNFB2F)などの良書が流通しているので、この章は飛ばしてそっちを読んだ方がいいと思います。

### SeleniumIDEを使用する例
ブラウザの操作を記憶する録画機能が提供されておりGUIベースで自動操作の処理を記載できます。録画した操作内容はスクリプトとして記録され、後で修正が可能です。また、別のプログラム言語で記載されたテストコードに変換することもできます。

以下はGoogle検索の操作をキャプチャした例になります。

https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/ed3262e8-84b7-732e-f25e-2abcd10144c3.gif

SeleniumIDEはChromeまたはFirefoxの拡張機能として提供されています。

「あれ？SeleniumIDEって終わったんじゃなかったっけ？」という人は下記の経緯を参照してください。

**Webブラウザ自動化ツール「Selenium IDE」の今までとこれから**
https://www.valtes.co.jp/qbookplus/509

#### ChromeでSeleniumIDEを使用する
(1)[Chrome用のSeleniumIDE](https://chrome.google.com/webstore/detail/selenium-ide/mooikfkahbdckldjjndioackbalphokd)を拡張機能として追加します。

(2)ブラウザの上部にSeleniumIDEのアイコンが表示されるのでクリックします。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/dc05f0e5-ef0e-ea35-d5ab-e854f0bffc97.png

(3)SeleniumIDEのポップアップが表示されるので「Record a new test in new project」を選択します。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/7d8c8966-f6cc-01ba-440e-4157b5a45104.png

(4)「Name your new project」ダイアログが表示されるので任意のプロジェクト名を入力して「OK」ボタンを押します。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/4a2058b6-eb8e-0396-b9ed-f3fafcd7c320.png

(5)「Set your projects's base URL」ダイアログが表示されるので操作元になるURLを入力します。たとえばGoogle検索をする例だと「[https://www.google.com](https://www.google.com)」を入力して「START RECORDING」ボタンを押下します。

https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/650993ef-cefe-3a1e-1b9f-450fdcfc0be4.png

(6)操作の記録が始まると右下に「Selenium IDE is recording...」と書かれた新しいブラウザが開きます。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/6c3edfac-3588-3946-720a-85b9cc489b1b.png
このブラウザを使用して記録したい任意の操作を行います。

(7)操作の記録を終了したい場合、「Selenium IDE」ウィンドウの右上の「Stop recording」アイコンを押します。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/3dd8367a-8a33-a9cf-f95b-8f4af8fff180.png

(8)「Name your new test」ポップアップが表示されるので任意のテスト名を入力して「OK」を押します。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/5fde2544-5c6b-9ff7-c60c-55f2712274a4.png


(9)SeleniumIDE ウィンドウに今回操作した内容がスクリプトとして記録されます。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/b596fef0-152c-6c72-92d5-9b5ff8ac320b.png

(10)記録したスクリプトは「Run Current Test」アイコンを押すことで再実行可能です。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/8808248c-4769-f72d-47b8-d2d2c6150c5f.png

また、JUnit や pytest、 JavaScript Mochaといった他のプログラミング言語のユニットテストとしてエクスポートすることが可能です。

#### FirefoxでSeleniumIDEを使用する
[Firefox用のSeleniumIDE](https://addons.mozilla.org/en-US/firefox/addon/selenium-ide/)を拡張機能として追加します。
あとの操作はChromeと同じです。

ただし、記録される操作はFirefoxとChromeで差異があります。
Firefoxの場合、ブラウザのスクロール操作が記録されていましたが、Chromeでは記録されていませんでした。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/f0e109c2-cbdc-342c-a9c2-52e8ef621449.png

なお、手で同じコマンドを追加すると、再生はChromeでも動作しました。

### プログラムからSeleniumを利用する
#### C#の場合
まず、NuGetでSelenium.WebDriverとSelenium.Supportに加えて操作したいブラウザのDriverを入手します。今回はChromeを操作したいので、Selenium.Chrome.WebDriverを入手します。

https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/72c841af-5b03-d6a3-e110-ec0289f5a43f.png

https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/24c7d9fc-c97a-70f1-f2ee-6bb5bffea816.png

C#でのSeleniumの操作例は以下のようになります。

```c#  
using OpenQA.Selenium;  
using OpenQA.Selenium.Chrome;  
using System;  
using System.IO;  
using System.Reflection;  
  
namespace chromeSample  
{  
    class Program  
    {  
        static void Main(string[] args)  
        {  
            using (var driver = new ChromeDriver(Path.GetDirectoryName(Assembly.GetEntryAssembly().Location)))  
            {  
                driver.Url = "http://needtec.sakura.ne.jp/auto_demo/form1.html";  
                driver.FindElementByName("name").SendKeys("名前太郎");  
                driver.FindElementByName("mail").SendKeys("test@co.jp");  
                driver.FindElementByName("comment").SendKeys("猫猫子猫\n\r犬犬子犬");  
  
                // チェックボックス  
                var chks = driver.FindElements(By.Name("q1[]"));  
                chks[0].Click();  
                chks[2].Click();  
  
                // オプションボタン  
                var opts = driver.FindElements(By.Name("men"));  
                opts[1].Click();  
  
                // 複数選択  
                var sel = new OpenQA.Selenium.Support.UI.SelectElement(driver.FindElement(By.Name("osi[]")));  
                sel.DeselectAll();  
                sel.SelectByIndex(1);  
                sel.SelectByIndex(2);  
  
                // 登録ボタン押下   
                driver.FindElement(By.XPath("//input[@value='登録する']")).Click();  
                // OKボタンを押す  
                var confirm = driver.SwitchTo().Alert();  
                confirm.Accept();  
  
                // 結果の出力  
                var results = driver.FindElementsByTagName("tr");  
                foreach(var rec in results)  
                {  
                    Console.WriteLine(rec.Text);  
                }  
                Console.ReadLine();  
                driver.Quit();  
            }  
        }  
    }  
}  
```

ページの切り替え時に待機処理をいれていませんが、暗黙的にタイムアウトまでDOMをポーリングしています。このタイムアウトについては下記を参照してください。

**The default value of timeouts on selenium webdriver**
https://stackoverflow.com/questions/30114976/the-default-value-of-timeouts-on-selenium-webdriver

#### PowerShellの場合
PowerShellでもC#と同様な実装が可能です。
下記のページのSelenium Client & WebDriver Language BindingsからC#のクライアントと操作したいブラウザのWebDriverをダウンロードしてください。

https://www.seleniumhq.org/download/

https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/61f80dfc-be45-eabd-1269-f6db15b3cd79.png

https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/7c411ab4-8e4c-56e3-f6ad-22fae35e9802.png

クライアントをダウンロードすると以下のようなファイルが入っています。

 - Selenium.Support.3.14.0.nupkg
 - Selenium.WebDriver.3.14.0.nupkg
 - Selenium.WebDriverBackedSelenium.3.14.0.nupkg

これは圧縮されているファイルなので拡張子をzipに変更すればDLLを取り出せます。
サポートしているバージョンが.NET3.5以上なのでWindows7に初期から入っているPowerShellでは動作しません。

PowerShellでのサンプルは以下のようになります。

```powershell  
# 以下参考  
# https://tech.mavericksevmont.com/blog/powershell-selenium-automate-web-browser-interactions-part-i/  
$dllPath = Split-Path $MyInvocation.MyCommand.Path  
Add-Type -Path "$dllPath\WebDriver.dll"  
Add-Type -Path "$dllPath\WebDriver.Support.dll"  
  
# chromedriver.exeがあるディレクトリを指定  
$driver = New-Object OpenQA.Selenium.Chrome.ChromeDriver("C:\tool\selenium\chromedriver_win32\")  
$driver.Url = "http://needtec.sakura.ne.jp/auto_demo/form1.html"  
$driver.FindElementByName("name").SendKeys("名前太郎")  
$driver.FindElementByName("mail").SendKeys("test@co.jp")  
  
# テキストエリア  
$comment = @"  
猫猫子猫  
犬犬子犬  
"@  
$driver.FindElementByName("comment").SendKeys($comment)  
  
# チェックボックス  
$chks = $driver.FindElements([OpenQA.Selenium.By]::Name("q1[]"))  
$chks[0].Click()  
$chks[2].Click()  
  
# オプションボタン  
$opts = $driver.FindElements([OpenQA.Selenium.By]::Name("men"))  
$opts[1].Click()  
  
# 複数選択  
$selElem = $driver.FindElement([OpenQA.Selenium.By]::Name("osi[]"))  
$sel = New-Object OpenQA.Selenium.Support.UI.SelectElement -ArgumentList $selElem  
$sel.DeselectAll();  
$sel.SelectByIndex(1);  
$sel.SelectByIndex(2);  
  
# ボタン押下  
$driver.FindElement([OpenQA.Selenium.By]::XPath("//input[@value='登録する']")).Click()  
$confirm = $driver.SwitchTo().Alert();  
$confirm.Accept()  
  
# 結果表示  
$results = $driver.FindElementsByTagName("tr");  
foreach($rec in $results)  
{  
    Write-Host $rec.Text  
}  
$driver.Quit()  
$driver.Dispose()  
  
Write-Host("OK")  
```


#### Pythonの場合
PythonでもSeleniumは使用可能です。まずpipコマンドでseleniumをインストールします。

```  
pip install -U selenium  
```

サンプルコードは以下のようになります。

```python  
from selenium import webdriver  
from selenium.webdriver.support.ui import Select  
  
driver = webdriver.Chrome("C:\\tool\\selenium\\chromedriver_win32\\chromedriver.exe")  
driver.get("http://needtec.sakura.ne.jp/auto_demo/form1.html")  
driver.find_element_by_name("name").send_keys("名前太郎");  
driver.find_element_by_name("mail").send_keys("test@co.jp");  
driver.find_element_by_name("comment").send_keys("猫猫子猫\n\r犬犬子犬");  
  
# チェックボックス  
chks = driver.find_elements_by_name("q1[]")  
chks[0].click()  
chks[2].click()  
  
# オプションボタン  
opts = driver.find_elements_by_name("men")  
opts[1].click()  
  
# 選択  
sel = Select(driver.find_element_by_name("osi[]"))  
sel.deselect_all()  
sel.select_by_index(1)  
sel.select_by_index(2)  
  
# ボタン押下  
driver.find_element_by_xpath("//input[@value='登録する']").click()  
driver.switch_to.alert.accept()  
  
# 結果  
results = driver.find_elements_by_tag_name("tr")  
for rec in results:  
    print(rec.text)  
  
driver.close()  
```

#### NodeJsの場合
NodeJsでもSeleniumの操作は可能です。npmコマンドを使用してseleniumをインストールします。

```  
npm install selenium-webdriver  
```

簡単なサンプルは以下のようになります。

```javascript  
// 以下参考  
// https://qiita.com/tonio0720/items/70c13ad304154d95e4bc  
// https://stackoverflow.com/questions/26191142/selenium-nodejs-chromedriver-path  
// https://seleniumhq.github.io/selenium/docs/api/javascript/index.html  
const webdriver = require('selenium-webdriver');  
const chrome = require('selenium-webdriver/chrome');  
const path = 'C:\\tool\\selenium\\chromedriver_win32\\chromedriver.exe';  
const service = new chrome.ServiceBuilder(path).build();  
chrome.setDefaultService(service);  
  
(async () => {  
  const driver = await new webdriver.Builder()  
                            .withCapabilities(webdriver.Capabilities.chrome())  
                            .build();  
  await driver.get('http://needtec.sakura.ne.jp/auto_demo/form1.html');  
  await driver.findElement(webdriver.By.name("name")).sendKeys("名前太郎");  
  await driver.findElement(webdriver.By.name("mail")).sendKeys("test@co.jp");  
  await driver.findElement(webdriver.By.name("comment")).sendKeys("猫猫子猫\n\r犬犬子犬");  
  
  // チェックボックス  
  let chks = await driver.findElements(webdriver.By.name("q1[]"));  
  await chks[0].click();  
  await chks[2].click();  
  
  // オプションボタン  
  let opts = await driver.findElements(webdriver.By.name("men"));  
  await opts[1].click();  
  
  // 複数選択  
  let sel = await driver.findElements(webdriver.By.xpath("//select[@name='osi[]']/option"));  
  await sel[1].click();  
  await sel[2].click();  
  
  // ボタン押下  
  await driver.findElement(webdriver.By.xpath("//input[@value='登録する']")).click();  
  await driver.switchTo().alert().accept();  
  
  // 結果取得  
  let results = await driver.findElements(webdriver.By.tagName("tr"));  
  for (let i = 0; i < results.length; ++i) {  
    console.log(await results[i].getText());  
  }  
  
  driver.quit();  
})();  
  
```

### Seleniumで既に起動しているブラウザの操作は行えるか？
Seleniumを介さず起動していたブラウザの自動操作については**公式にはサポートしていません。**
下記に幾つかの回避法が紹介されていますが、あくまで非公式の内容になります。

**Can Selenium interact with an existing browser session?**
https://stackoverflow.com/questions/8344776/can-selenium-interact-with-an-existing-browser-session

#### Chromeのremote-debugging-portを使用する例
remote-debugging-portを指定して起動したChromeに対しての自動操作の例が以下のページで紹介されています。

**How to connect Selenium to an existing browser that was opened manually?**
https://cosmocode.io/how-to-connect-selenium-to-an-existing-browser-that-was-opened-manually/

1. まずChromeを以下のオプションを付けて起動してください。

```  
"C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222 --user-data-dir=C:\dev\python3\selenium\tmp  
```

2. 起動したChromeの画面を操作します。今回の例ではTwitterの画面にログインしたと仮定します。
3. その後、下記の操作を行うことで2で起動中の画面を使用してツイートが可能です。

```python  
from selenium import webdriver  
from selenium.webdriver.chrome.options import Options  
  
# https://cosmocode.io/how-to-connect-selenium-to-an-existing-browser-that-was-opened-manually/  
# TODO remote-debugging-portを指定してChromeを起動後、「https://twitter.com/home」を表示  
# "C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222 --user-data-dir=C:\dev\python3\selenium\tmp  
chrome_options = Options()  
chrome_options.add_experimental_option("debuggerAddress", "127.0.0.1:9222")  
# Change chrome driver path accordingly  
chrome_driver = "C:\\tool\\selenium\\chromedriver_win32\\chromedriver.exe"  
driver = webdriver.Chrome(chrome_driver, chrome_options=chrome_options)  
  
driver.find_element_by_xpath("//a[@aria-label='ツイート']").click();  
driver.find_element_by_xpath("//div[@aria-label='テキストをツイート']").send_keys('あいうえお')  
driver.find_element_by_xpath("//div[@data-testid='tweetButton']").click();  
  
# Chromeを閉じたい場合  
# driver.close()  
  
```

### Seleniumを使用する方法のまとめ
Seleniumを使用することで様々なブラウザを様々なプログラミング言語で操作できることができます。
またSeleniumIDEを使用することでプログラミングをせずにブラウザの自動操作がおこなえます。

ただし、Flashページなどの画像認識を必要とする操作の場合は他の方法を検討してください。
たとえば以下のような方法があります。

**Sikulix1.1.4を使って画面の自動操作をする**
https://qiita.com/mima_ita/items/8f653042ac9140e5023f

**C#やPowerShellで画面上の特定の画像の位置をクリックする方法**
https://qiita.com/mima_ita/items/f7d2c38767bda8b35cbd

## 拡張機能を作成する方法
ChromeやFirefoxで利用できる拡張機能を使用して表示中のページを自動操作することが可能です。

**Chromeの拡張機能**
https://developer.chrome.com/extensions

**Firefoxの拡張機能**
https://developer.mozilla.org/ja/docs/Mozilla/Add-ons/WebExtensions/Your_first_WebExtension

以下はChromeの拡張機能を使用してページを自動操作後、操作結果をメッセージボックスで表示しています。

https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/3f0b24c8-533d-c08c-b4e1-0d72ab48160d.gif

**操作対象のページ**
http://needtec.sakura.ne.jp/auto_demo/form1.html

拡張機能を使った自動操作の仕組みは下記の通りです。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/5536e030-d427-504a-3fa2-8f6630e8403f.png

default_popupからcontent_scriptsに対して自動操作の開始指示をメッセージを使用して行います。
入力ページのcontent_scriptsは項目の入力とボタンの押下を行います。
出力ページのcontent_scriptsは出力ページの内容を取得してメッセージを使用してdefault_popupに内容を送信します。

なお、ChromeとFirefoxの拡張機能は同じような実装で作成できます。


### Chromeの拡張機能で自動操作

**Chrome拡張機能での自動操作のサンプル**
https://github.com/mima3/auto_sample/tree/master/auto_chrome_sample

以下が実際の自動操作を行っているcontent_scriptになります。

```javascript:content_input.js  
chrome.runtime.onMessage.addListener(  
  function(request, sender, sendResponse) {  
    console.log(request);  
    let nameElem = document.getElementsByName('name');  
    nameElem[0].value = '名前太郎';  
    let mailElem = document.getElementsByName('mail');  
    mailElem[0].value = 'test@co.jp';  
    let commentElem = document.getElementsByName('comment');  
    commentElem[0].value = '猫猫子猫\n犬犬子犬';  
    // チェックボックス  
    let chkElem = document.getElementsByName('q1[]');  
    chkElem[0].click();  
    chkElem[2].click();  
    // ラジオボタン  
    let radioElem = document.getElementsByName('men');  
    radioElem[1].click();  
    // 選択項目  
    var itr = document.evaluate("//select[@name='osi[]']/option", document, null, XPathResult.UNORDERED_NODE_ITERATOR_TYPE, null );  
    var node = itr.iterateNext();  
    while(node) {  
      if (node.textContent === '千鶴さん' || node.textContent === 'さおりん') {  
        node.selected = true;  
      }  
      node = itr.iterateNext();  
    }  
    // ボタン押下  
    // contents.jsでwindows.confirmを書き換えてもブラウザ側の処理に影響を与えない  
    // そのため、window.confirmを書き換えるscritpをタグとして挿入する  
    // 考え方は以下を参考  
    // https://qiita.com/suin/items/5e1aa942e654bce442f7  
    let scr = document.createElement("script");  
    scr.setAttribute('type', 'text/javascript');  
    scr.innerText = 'window.confirm = function () { return true; }';  
    document.body.appendChild(scr);   
    setTimeout(function(){   
      //ここでやってもブラウザ上のwindow.confirmは影響ない。  
      var btnElem = document.evaluate("//input[@value='登録する']", document, null, XPathResult.FIRST_ORDERED_NODE_TYPE, null);  
      btnElem.singleNodeValue.click();  
    }, 0);  
  }  
);  
```

JavaScriptのDOM操作で入力項目を設定後、登録ボタンを押下します。
この際、confirmで確認ダイアログが表示されるため、window.confirmの処理を上書きしてダイアログが出ないようにしています。
content_scriptからブラウザで使用しているJavaScriptを更新するため、scriptタグを埋め込んでいます。
この考え方は下記を参考にしました。

**Chrome拡張開発: 拡張からページにJavaScriptを送り込みたい**
https://qiita.com/suin/items/5e1aa942e654bce442f7


### Firefoxの拡張機能で自動操作

**Firefox拡張機能での自動操作のサンプル**
https://github.com/mima3/auto_sample/tree/master/auto_firefox_sample

### 拡張機能での自動操作のまとめ
ブラウザの拡張機能を利用することでブラウザの自動操作が行えます。
この方法のメリットとしてはインターネットの接続がなくてもテキストエディタのみで自動操作を行うためのスクリプトが作成できることです。（ブラウザを開発者モードで動かしていいという条件は必要）

もし、自動操作中にネイティブのアプリと連携が必要になった場合は[Native Messaging](https://developer.chrome.com/apps/nativeMessaging)を使用してください。このNaitiveMessageを使用したサンプルは以下にあります。

 - [IEが死んだ世界 chromeの拡張機能であるnative-messageを使用した回避案](https://qiita.com/mima_ita/items/7f1c7191045db7290a4e#chrome%E3%81%AE%E6%8B%A1%E5%BC%B5%E6%A9%9F%E8%83%BD%E3%81%A7%E3%81%82%E3%82%8Bnative-message%E3%82%92%E4%BD%BF%E7%94%A8%E3%81%97%E3%81%9F%E5%9B%9E%E9%81%BF%E6%A1%88)

## RPAツールを使用する方法
お高いRPAツールはブラウザの操作をサポートしている商品が多いです。
今回は小規模事業や個人利用なら無料でしようできる[UiPath Community](https://www.uipath.com/platform-trial)を利用してChromeの操作を行います。

ChromeをUiPathで操作する場合、Chromeの拡張機能をインストールする必要があるので、下記を参考にインストールしてください。
https://docs.uipath.com/studio/lang-ja/docs/installing-the-chrome-extension

(1)UIPathで新規プロジェクトを作成します。言語はC#を選択します。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/65d75f18-5d34-d17a-0519-fdfb0a4f7c87.png

(2)「ブラウザを開く」アクティビティを追加します。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/cdd293dc-6fd6-96be-7102-a90950d2ff56.png

|プロパティ|値|
|:---------|:----------|
|url|"http://needtec.sakura.ne.jp/auto_demo/form1.html"|
|ブラウザの種類|Chrome|

(3)「文字を入力」アクティビティを追加して「画面上で指定」でブラウザ上のテキスト入力項目を指定します。

https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/c6a9feff-b815-135d-da36-d3c34adb281f.png

https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/9b51d660-ae95-88f9-a294-f9408d55abf2.png

https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/5ccec345-a813-c46a-430a-fe2f57c0d2c6.png

(4)「文字を入力」アクティビティのプロパティを設定します。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/3b9123aa-b99a-0fe8-f240-d6acc05c110e.png

(5)(3)～(4)を繰り返して「名前：」、「メールアドレス：」、「コメント：」を入力します。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/16eae779-b88c-da88-dae4-72d4e1baaff2.png

|プロパティ|値|
|:---------|:----------|
|表示名|文字を入力 'INPUT-名前'|
|テキスト|"名前太郎"|
|フィールド内を削除|ON|
|ウィンドウメッセージを送信|OFF|
|入力をシミュレート|OFF|

|プロパティ|値|
|:---------|:----------|
|表示名|文字を入力 'INPUT-メールアドレス'|
|テキスト|"test@mail.co.jp"|
|フィールド内を削除|ON|
|ウィンドウメッセージを送信|**ON** ※デフォルトの挙動だとIMEが有効となり全角で入力されてしまう|
|入力をシミュレート|OFF|

|プロパティ|値|
|:---------|:----------|
|表示名|文字を入力 'TEXTAREA-コメント'|
|テキスト|"猫猫子猫\n\r犬犬子犬"|
|フィールド内を削除|ON|
|ウィンドウメッセージを送信|OFF|
|入力をシミュレート|OFF|


(6)「クリック」アクティビティを追加して「画面上で指定」でブラウザ上のクリックが必要な項目をを指定します。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/473f4db9-49cc-ceea-65a8-71246ecacb51.png

https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/eec62f86-8f3c-7c5c-dc81-52eb1119da8f.png

https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/69d5abf8-8245-0b76-c381-e47a85da29e5.png

(7)「クリック」アクティビティのプロパティを設定します。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/4227821f-810a-d51a-73e8-b47d4c245586.png

(8)(5)～(6)を繰り返して「その1」、「その3」、「そば」をクリックします。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/8b50bca6-fa55-69eb-fe4c-3f172bbaf8c8.png

|プロパティ|値|
|:---------|:----------|
|表示名|クリック 'INPUTーその１'|
|ウィンドウメッセージを送信|**ON**(デフォルトだと挙動が安定しない)|
|入力をシミュレート|OFF|

|プロパティ|値|
|:---------|:----------|
|表示名|クリック 'INPUTーその3'|
|ウィンドウメッセージを送信|**ON**(デフォルトだと挙動が安定しない)|
|入力をシミュレート|OFF|

|プロパティ|値|
|:---------|:----------|
|表示名|クリック 'INPUTーそば'|
|ウィンドウメッセージを送信|**ON**(デフォルトだと挙動が安定しない)|
|入力をシミュレート|OFF|

(9)リストを複数選択するために「JSスクリプトを挿入」アクティビティを追加します。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/4a56143f-5f4d-120e-377b-4625c3d41dfd.png

選択したJSスクリプトは下記の通りです。

```javascript:select_multi.js  
function selectmulti(e, aryStr) {  
  var ary = JSON.parse(aryStr);  
  var itr = document.evaluate("//select[@name='osi[]']/option", document, null, XPathResult.UNORDERED_NODE_ITERATOR_TYPE, null );  
  var node = itr.iterateNext();  
  while(node) {  
    if (ary.indexOf(node.textContent) >= 0) {  
      node.selected = true;  
    }  
    node = itr.iterateNext();  
  }  
}  
```

|プロパティ|値|
|:---------|:----------|
|スクリプトコード|select_multi.js|
|入力パラメータ|"[\"千鶴さん\",\"さおりん\"]"|

CTRLキーを押しながらのリスト項目をクリックする操作や、「複数の項目を選択」アクティビティを使用した実装だと動作しない場合がありました。

**Web上のリストボックスで複数選択したい**
https://forum.uipath.com/t/web/113531/9

また、ここで指定したJavaScript中で日本語やハングルは使用しないでください。文字化けします。日本語などが必要な場合は引数で渡すようにしてください。
たとえば「alert("千鶴さん");」とかいうコードを埋め込むと以下のようになります。

https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/e778ed30-a9e5-faf9-d0e3-343df2a8b5c8.png

(10)登録ボタンを押下するために「クリック」アクティビティを追加します。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/6cb8f313-e779-4cec-3aea-2830c05bfcb3.png

|プロパティ|値|
|:---------|:----------|
|表示名|クリック 'INPUT-登録'|
|ウィンドウメッセージを送信|**ON**(デフォルトだと挙動が安定しない)|
|入力をシミュレート|OFF|

(11)登録ボタン押下後の確認メッセージを閉じるために「画像をクリック」アクティビティを追加します。

https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/949ba59b-9407-0b90-ac78-45ff92681fd6.png

なお、「select_multi.js」に以下のコードを追加してconfirm関数を上書きして確認メッセージを表示させないことも可能です。

```javascript  
  let scr = document.createElement("script");  
  scr.setAttribute('type', 'text/javascript');  
  scr.innerText = 'window.confirm = function () { return true; }';  
  document.body.appendChild(scr);  
```

(12)登録後のページのデータを取得するために「データスクレイピング」を行います。

**「データスクレイピング」アイコンを押下します。**
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/bff4d42b-642c-6db6-ed16-7e5f82035706.png

**「取得ウィザード」で「次へ」ボタンを押下します。**
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/d793f8da-d3d9-914e-de8b-61001419f4be.png

**要素の選択が可能になるのでテーブルのセルを選択します。**
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/eeac4f2e-3887-1ee6-c756-310f805077de.png

**「表全体からデータを抽出しますか？」の確認メッセージには「はい」を選択します。**
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/8e733b2c-99cb-0b9f-5615-3d6150a49183.png


**「取得ウィザード」で「終了」ボタンを押下します。**
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/60071a2f-293f-2aa2-8cff-8f25af187efd.png


**「次へのリンクを指定」の確認メッセージには「いいえ」を選択します。**
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/b682adba-93ef-5c84-e1ee-b21942b08508.png

**「データスクレイピング用のアクティビティが追加されます。**
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/31fd62c2-f764-df3a-1c1a-4d7cadc08070.png

(13)「構造化データを抽出」アクティビティの「出力」プロパティに対してCTRL+Kを押下してresult変数を追加します
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/d1345e95-da3e-28e3-e2f2-80314c22a9b9.png
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/df45f824-a78a-8dce-3d67-62fe348854af.png

(14)「繰り返し（各行）」アクティビティを追加します。この際、コレクションには「result」変数を指定してください。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/84aafb6e-a409-a117-1948-65a3e4c74e1d.png

(15)「繰り返し（各行）」アクティビティに「一行を書き込み」アクティビティを追加します。

https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/a4a2049b-b505-3152-1c34-770b7050c074.png

|プロパティ|値|
|:---------|:----------|
|Text|row[0].ToString() + " " + row[1].ToString()|

(16)これまでの操作を再生すると以下のようになります。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/188c0f1d-366a-d717-e164-30ea47a421ee.gif


### UiPathでのブラウザの自動操作のまとめ
UiPathを使用したメリットは以下の通りです。
・要素を画面から選択できる
・画像認識による自動操作ができる
・ブラウザ以外の自動操作が同じ操作感で行える。
・今回は説明してませんが[UiPath Orchestrator](https://kashika.biz/uipath_orchestrator_01/)で資産管理が容易になる

逆にデメリットは以下の通りです。
・GUIでのプログラミングになるので、複雑な実装が困難である
・GUIなので変更点の差分を見るのが困難で、コードレビューが負担になる。※結局はxamlなのでテキストで差分はとれるが…
・なれないとハマるポイントが多い。
・UiPathの操作でDOMの要素を変更したりしているのでシステム試験等で使用する場合、妥当性を考える必要がある。
例：UiPathで操作した要素には以下のように「uipath_custom_id」という属性が追加されている。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/15ba6734-9227-a088-106a-53de6a32a1ef.png

UiPathは外部プログラムを呼び出す機能やPowerShellの実行が可能なのでブラウザの操作は別の手法で行うことも可能です。


なお、RPAで未経験者でもお手軽自動操作とかいう言説が大きくなっていますが、正直、WebアプリケーションやWindowsアプリケーションを組んだことのない人が簡単に使えると言われると大きな疑問が残ります。
逆にRPAツール不要論もありますが、UiPath Orchestratorの存在や、簡単なフローが頻繁に変わる業務形態における仕事の分担という観点で、そのRPA不要論についても絶対的な真理とは言えないでしょう。

状況にあわせた組み合わせが必要と思います。

## ブラウザのGUI上の操作をプログラム上で真似して自動化する方法のまとめ
ここまでで、ブラウザのGUI上の操作をプログラム上で真似して自動化する方法について紹介しました。

RPAツールを使える環境の場合、RPAツールは自動化を行う上で便利ではありますが、それを使うことを目的にしない方がいいです。必要に応じて別の方法をミックスして使うようにしましょう。

外部ライブラリを使用できる環境の場合、Seleniumを採用するのが一番楽だと思います。ChromeやFirefoxならSeleniumIDEで録画機能もついているので生産性は高いでしょう。

外部ライブラリが使用できない環境の場合、InternetExploreのCOM又は、ブラウザの拡張機能を使用することになります。
IEを使用する場合、操作対象のWebアプリケーションのサポート状況をよく確認しましょう。

いずれの方法でブラウザ操作を自動化するにせよ以下の点は気をつけてください。

### 安易なSleepを使用しない
たとえば、適当に2秒待つという処理をいれた場合、ネットワークやPCの負荷状況によって動作しない可能性があります。
Sleepよりも以下で判断するようにしましょう。

 ・ document.readyStatusなどを活用する
 ・ 特定の要素が出現したか、消えたかを見て判断する。
　→SeleniumやUiPathでは、要素の出現消滅の検出をサポートする機能が提供されている

### テキスト入力は手動入力と異なる挙動をする可能性がある
テキスト入力を行う場合、手動で入力した場合と異なる挙動をする可能性があります。
たとえばキーボード操作のイベントで何らかの処理していたり、フォーカスの移動で何らかの処理をしていたりする場合です。
UiPathの場合、入力モードに以下の３種類があるので必要に応じて使いわけてください。

・デフォルト：デバイスドライバ経由なので手入力にもっとも近い
・WindowsMessage：Windowsのメッセージを利用してテキストを入力している。
・Simulate：コントロールを直接操作している

それ以外の場合は、JavaScriptのコード上でアプリが期待するイベントを無理やり起こす必要があります。

### 必要に応じてJavaScriptを利用する
複雑なUIの場合、画面要素を一々クリックするより、WebアプリケーションのJavaScriptを直接実行した方が早い場合があります。
また、いままでの例にでてきたように、alertやconfirmのような自動操作がし辛いポップアップの出現を抑止することが可能になります。

### テストの自動化についてはテスト方針をよく確認する
ツールをつかったテストは強力ですが、それはユーザが動かしたものと全く同一にならないことに注意してください。
たとえば、先にあげたJavaScriptを呼び出して処理を行った場合、それがテストとして妥当かどうかはテストの方針や観点しだいになります。

### UIの軽微な変更で動作しなくなることを忘れないこと
ブラウザの自動テストはUIの軽微な変更で簡単に動かなくなります。
たとえばリストの2番目と3番目の項目を選択するという実装だと、リストの項目が追加された場合に簡単に動作しなくなります。
これがなるべく影響を受けないような書き方をすることも可能ですが、限界はあります。

もしブラウザの自動化スクリプトを重要な業務で使用している場合は、前に動いたスクリプトだからと安心せずWebアプリケーションの変更にともなって定期的に以前に書いた自動化スクリプトが動作するか確認するようにしてください。


# ブラウザから送信しているデータを真似する方法
## ブラウザの送受信データの確認方法
ブラウザから送信しているデータを真似して自動化する前にブラウザからどんなデータを送受信しているか調べる方法を説明します。

下記のページで登録ボタンを押した場合にどのようなデータを送信しているか確認してみましょう。
http://needtec.sakura.ne.jp/auto_demo/form1.html

### Chromeでの送受信データの確認方法
(1)F12で開発者ツールを開き、「Network」タブを選択します。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/15507c7d-8cd1-d9be-1d6f-4dd6736043ea.png

(2)画面上で入力操作を行い「登録する」ボタンを押します。しばらくすると受信ファイルの一覧が表示されます。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/9b42edb9-3185-4d5e-4c5a-6b7fdda8b054.gif

(3)「regist1.php」などの受信ファイルを選択後に、Headersタブを選択すると送信データが確認できます。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/4d467b8b-0250-0eb7-2ff3-8ef8bae8ab6d.png


(4)Responseタブを選択すると受信内容が確認できます。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/a84dee84-ed3a-cf16-2875-e18ffb5b915d.png

### Firefoxでの送受信データの確認方法
(1)F12で開発者ツールを開き、「ネットワーク」タブを選択します。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/43273a1b-90ac-1812-ad0a-1182739afd3d.png

(2)画面上で入力操作を行い「登録する」ボタンを押します。しばらくすると受信ファイルの一覧が表示されます。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/32a644dd-b2e5-151b-5aa9-18dffcf64fdc.png

(3)「regist1.php」などの受信ファイルを選択します。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/a829ed9d-7968-8bd3-a7e9-839bb29ddbd7.png

(4)パラメータタブでFormの送信情報を確認できます。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/90e7d715-6c91-59bb-75c9-0a9f824a35b6.png

(5)応答タブでサーバーからのレスポンスデータを確認できます。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/cf4dec61-3dc6-cba0-6c63-10d2e028d2dd.png

### IE11での送受信データの確認方法
(1)F12で開発者ツールを開き、「ネットワーク」タブを選択します。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/25b4bfde-7e44-fab7-5b7b-cfa285f7ca16.png

(2)画面上で入力操作を行い「登録する」ボタンを押します。しばらくすると受信ファイルの一覧が表示されます。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/3761684f-c688-f6f9-0d40-3520b825f2f3.png

(3)「regist1.php」などの受信ファイルを選択します
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/42722618-cf31-c93c-ac69-9c3adf4e1ca2.png

(4)本文タブを選択し、さらに「要求本文」タブを選択するとFormの送信情報を確認できます。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/fd102b1f-98b8-ea22-0206-ef1db789e9fa.png


(5)本文タブを選択し、さらに「応答本文」タブを選択するとサーバーからのレスポンスデータを確認できます。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/3c278474-df84-0892-faba-a48cf35d02b5.png

### 旧Edgeでの送受信データの確認方法
(1)F12で開発者ツールを開き、「ネットワーク」タブを選択します。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/4c07b923-faa5-72f8-7c0c-d6694b36206e.png

(2)画面上で入力操作を行い「登録する」ボタンを押します。しばらくすると受信ファイルの一覧が表示されます。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/ca368486-e502-b0d5-ec83-9ec640e5613c.png

(3)「regist1.php」などの受信ファイルを選択します
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/372ebbee-def6-5fb1-d210-af6c45adc80e.png

(4)本文タブを選択し、さらに「要求本文」タブを選択するとFormの送信情報を確認できます。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/aae35af1-5e5c-836d-92dc-08fa98771969.png

(5)本文タブを選択し、さらに「応答本文」タブを選択するとサーバーからのレスポンスデータを確認できます。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/b31569dc-4e74-258f-fe18-c34fc47a2e9d.png

### 新Edgeでの送受信データの確認方法
Chromeと同じです。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/783ea38d-3afb-9c24-c0b7-a0de87f46a12.png

## 単純なFormデータの送信例
下記のページのような単純なフォームのデータの送信例を説明します。
http://needtec.sakura.ne.jp/auto_demo/form1.html

### curlコマンド
macやlinux系のOSなら[curlコマンド](https://curl.haxx.se/docs/manpage.html)を使用することで単純なフォームデータをPOSTすることが可能です。
[windos10でもプリインストールされるようになった](https://moyapro.com/2019/04/18/windows10-curl-pre-install/)ようですが、文字コードの問題があるので注意が必要です。また、PowerShellを使用しているとcurlコマンドが利用できますが、これはInvoke-WebRequestの別名です。

以下はCentOS7でcurlコマンドを実行した例となります。

```  
>curl  -F "name=名前太郎" -F "mail=test@co.jp" -F "comment=コメント" -F "q1[]=その1" -F "q1[]=その3"  -F "men=soba" -F "osi[]=千鶴さん" -F "osi[]=さおりん"   http://needtec.sakura.ne.jp/auto_demo/regist1.php  
<html>  
<head>  
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />  
<title>sample</title>  
</head>  
<body>  
<table border="1">  
  <tr>  
    <td>名前</td><td>名前太郎</td>  
  </tr>  
  <tr>  
    <td>メールアドレス</td><td>test@co.jp</td>  
  </tr>  
  <tr>  
    <td>コメント</td><td>コメント</td>  
  </tr>  
  <tr>  
    <td>チェック</td>  
    <td>  
        その1, その3<br>    </td>  
  </tr>  
  <tr>  
    <td>めん</td><td>soba</td>  
  </tr>  
  <tr>  
    <td>おし</td><td>千鶴さん,さおりん</td>  
  </tr>  
</table>  
</body>  
</html>  
```


### powershellの例
PowerShellでは[Invoke-WebRequest](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/invoke-webrequest?view=powershell-5.1)を利用してFormデータを送信可能です。

```powershell  
	$data = @{  
	  name='名前太郎';  
	  mail='test';  
	  comment=@"  
	猫猫子猫  
	犬犬子犬  
"@;  
	  'q1[0]'='その1';  
	  'q1[1]'='その3';  
	  men='soba';  
	  'osi[0]'='千鶴さん';  
	  'osi[1]'='さおりん'  
	}  
	$ret = Invoke-WebRequest http://needtec.sakura.ne.jp/auto_demo/regist1.php -Method POST -Body $data -ContentType "application/x-www-form-urlencoded"  
	$html = $ret.ParsedHtml  
	$list = $html.getElementsByTagName("tr")  
	for($i=0; $i -lt $list.length; $i++) {  
	  $item = $list[$i]  
	  Write-Host($item.innerText)  
	  [System.Runtime.Interopservices.Marshal]::ReleaseComObject($item) | Out-Null  
	  Remove-Variable item -ErrorAction SilentlyContinue  
	}  
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($list) | Out-Null  
	Remove-Variable list -ErrorAction SilentlyContinue  
  
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($html) | Out-Null  
	Remove-Variable html -ErrorAction SilentlyContinue  
  
	[System.GC]::Collect()  
	[System.GC]::WaitForPendingFinalizers()  
	[System.GC]::Collect()  
```

ParsedHtmlはmshtmlになっているので、構文解析を容易に行えます。
なお、mshtmlはCOMなのでReleaseComObjectを実施して解放処理をしておいた方が無難です。

なお、ページによっては文字化けする場合があります。この場合は以下のように文字コードを変換して出力します。

```powershell  
# 以下参考  
# https://qiita.com/zaki-lknr/items/1ae3258d7b77c5e2a2ba  
$ret = Invoke-WebRequest http://needtec.sakura.ne.jp/auto_demo/form1.html  
$content = [System.Text.Encoding]::UTF8.GetString( [System.Text.Encoding]::GetEncoding("ISO-8859-1").GetBytes($ret.Content) )  
Write-Host $content  
```

### VBAまたはVBSの場合
[MSXML2.XMLHTTP](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/ms759148(v%3dvs.85)?ranMID=24542&ranEAID=je6NUbpObpQ&ranSiteID=je6NUbpObpQ-.nTP.GKYEngxVvAZbklXFg&epi=je6NUbpObpQ-.nTP.GKYEngxVvAZbklXFg&irgwc=1&OCID=AID2000142_aff_7593_1243925&tduid=(ir__19oa1xfwk0kfrlkrkk0sohzx0m2xghgc2yhm6a1a00)(7593)(1243925)(je6NUbpObpQ-.nTP.GKYEngxVvAZbklXFg)()&irclickid=_19oa1xfwk0kfrlkrkk0sohzx0m2xghgc2yhm6a1a00)又は[Windows HTTP](https://docs.microsoft.com/en-us/windows/win32/winhttp/winhttp-start-page)を利用することでFormデータを送信可能です。

受信したHTMLはMSHTML.HTMLDocumentできます

以下はVBSのサンプルになっていますがWScript.EchoをDebug.Print等に置き換えることでVBAでも流用できると思います。

**MSXML2.XMLHTTPの例**

```vb  
' 参考：  
' https://outofmem.tumblr.com/post/63052619242/vbaexcel-vba%E3%81%A7http%E9%80%9A%E4%BF%A1  
' https://stackoverflow.com/questions/9931429/parse-html-file-using-mshtml-in-vbscript  
Dim httpReq  
Set httpReq = CreateObject("MSXML2.XMLHTTP")  
  
Call httpReq.Open("POST", "http://needtec.sakura.ne.jp/auto_demo/regist1.php", False)  
Call httpReq.setRequestHeader("Content-Type", "application/x-www-form-urlencoded;charset=utf-8")  
Dim postData  
postData = "name=名前太郎&mail=test@co.jp&comment=猫猫" & vbCrLf & "子犬&q1[]=その1&q1[]=その2&men=soba&osi[]=千鶴さん&osi[]=さおりん"  
Call httpReq.Send(postData)  
  
Dim objHtml  
Set objHtml = CreateObject("htmlfile")  
call objHtml.write(httpReq.responseText)  
  
Dim list  
Set list = objHtml.getElementsByTagName("tr")  
Dim item   
For Each item In list  
  WScript.Echo item.innerText  
Next  
  
```

**Windows HTTPの例**

```vb  
'略  
Set httpReq = CreateObject("WinHttp.WinHttpRequest.5.1")  
'略  
```

CreateObjectに与える値を変更するだけで切り替えが可能です。
MSXML2.XMLHTTPとWindows HTTPで挙動に違いは**この時点**ではありません。(後述のCookieを使用する場合に違いがでてきます。)
この違いについては下記で話題にあがっています。　

 - [HTTP Requests in VBA: WinHttp, or MSXML2, or ...?](https://social.msdn.microsoft.com/Forums/sqlserver/en-US/a7445768-4e81-4c90-99ad-8173005f2d0c/http-requests-in-vba-winhttp-or-msxml2-or-?forum=windowsgeneraldevelopmentissues)

### Pythonの例
[http.client](https://docs.python.org/ja/3/library/http.client.html)を使用してFormデータを送信して結果を受信可能です。[html.parser](https://docs.python.org/3/library/html.parser.html)を利用することでHTMLの解析も行えます。おそらく、python3xが入っている環境ならどこでも使えると思います。

```python  
import http.client, urllib.parse  
from html.parser import HTMLParser  
  
# 結果ページを解析するパーサー  
class ResultParser(HTMLParser):  
  def __init__(self):  
      HTMLParser.__init__(self)  
      self.flag = False  
  
  def handle_starttag(self, tag, attrs):  
      if tag == "td":  
          self.flag = True  
  
  def handle_data(self, data):  
      if self.flag:  
          print (data)  
          self.flag = False  
  
conn = http.client.HTTPConnection('needtec.sakura.ne.jp')  
  
headers = {  
  'Content-type': ' application/x-www-form-urlencoded'  
}  
  
data = {  
  'name': '名前太郎',  
  'mail': 'test@co.jp',  
  'comment' : '猫猫子猫\n\r犬犬子犬',  
  'q1[0]' : 'その1',  
  'q1[1]' : 'その3',  
  'men' : 'soba',  
  'osi[0]' : '千鶴さん',  
  'osi[1]' : 'さおりん'  
}  
# r = requests.post('http://needtec.sakura.ne.jp/auto_demo/regist1.php', data=data)  
# print(r)  
# print(r.text)  
  
params = urllib.parse.urlencode(data)  
  
conn.request('POST', '/auto_demo/regist1.php', params, headers)  
response = conn.getresponse()  
print(response.status, response.reason)  
parser = ResultParser()  
# trの内容を出力  
parser.feed(response.read().decode())  
conn.close()  
```

#### 外部ライブラリを使う場合
[requestsパッケージ](https://pypi.org/project/requests/2.7.0/)を使うとデータの送受信が、[Beautiful Soup](https://www.crummy.com/software/BeautifulSoup/bs4/doc)を使うとHTMLの解析が楽になります。

```python  
import requests  
from bs4 import BeautifulSoup  
  
data = {  
  'name': '名前太郎',  
  'mail': 'test@co.jp',  
  'comment' : '猫猫子猫\n\r犬犬子犬',  
  'q1[0]' : 'その1',  
  'q1[1]' : 'その3',  
  'men' : 'soba',  
  'osi[0]' : '千鶴さん',  
  'osi[1]' : 'さおりん'  
}  
r = requests.post('https://needtec.sakura.ne.jp/auto_demo/regist1.php', data=data)  
print(r.status_code, r.reason)  
soup = BeautifulSoup(r.text)  
for tr in soup.find_all('tr'):  
  print('------------------------')  
  print(tr.text)  
```

## 認証があるページの例
単純なフォームの送信例はPOSTを一回送信するだけで済みましたが、認証処理やページの不正遷移防止が行われているWebアプリについてはサーバからの情報を受け取ってそれを基にデータを送信する必要があります。

今回は[bitnamiから取得できるRedmineのVM](https://bitnami.com/stack/redmine/virtual-machine)でチケット登録を行うサンプルを見てみます。
VMのもろもろの設定は[TestLinkで設定したときと同様に行えます。](https://qiita.com/mima_ita/items/ed56fb1da1e340d397b9#%E4%BB%AE%E6%83%B3%E7%92%B0%E5%A2%83%E3%81%AE%E8%AA%AC%E6%98%8E%E3%81%A8%E8%A8%AD%E5%AE%9A)

Redmineでログインしてチケットを登録するには以下の手順を踏む必要があります。

 - ログインページを取得する。
 - サーバーはヘッダーにセッションID、HTML中に認証トークン文字を埋め込んでログイン用のページを返す。
 - セッションIDをリクエストヘッダに設定し、リクエストボディに認証トークン、ユーザ名、パスワード、ログイン後の遷移ページ（チケット登録画面）を指定してログイン処理を行う。
 - サーバーはログインに成功したらチケット登録画面を返す。この際、認証トークン文字が新しいものに変更される。
 - セッションIDをリクエストヘッダに設定し、リクエストボディに認証トークン、チケット情報を指定してチケット登録処理を行う。


### PowerShellの例
PowerShellでRedmineのチケットを登録するには以下のようになります。

```powershell  
# エラーが起きたらとめる  
$ErrorActionPreference = "Stop"  
  
# サーバから取得したCookieの値からキーを指定して値を取得する  
function get_key_value($value, $key) {  
  $tmp = $value.substring($value.indexof($key) + $key.length)  
  $ret = $tmp.substring(0, $tmp.indexof(';'))  
  return $ret  
}  
  
# DOMを解析して指定の名前の指定の属性を取得する  
function get_attribyte_value($html, $elem_name, $attr_name) {  
  $elems = $html.getElementsByName($elem_name)  
  $elem = $elems[0]  
  $attrs = $elem.attributes  
  $attr = $attrs[$attr_name]  
  $ret = $attr.value  
  [System.Runtime.Interopservices.Marshal]::ReleaseComObject($elems) | Out-Null  
  [System.Runtime.Interopservices.Marshal]::ReleaseComObject($elem) | Out-Null  
  [System.Runtime.Interopservices.Marshal]::ReleaseComObject($attrs) | Out-Null  
  [System.Runtime.Interopservices.Marshal]::ReleaseComObject($attr) | Out-Null  
  return $ret  
}  
  
##################################  
# Redmineの設定値  
##################################  
$redmine_host = "192.168.0.200"  # サーバ名  
$redmine_project = "test1"       # プロジェクト名  
$username = "user"  
$password = "pass"  
  
##################################  
# ログインページを初回アクセスしてセッションIDとcsrf-tokenを取得する  
##################################  
$ret = Invoke-WebRequest "http://$redmine_host/login" -Method GET  
  
# セッションID取得  
$cookie = $ret.Headers['Set-Cookie']  
$session_id = get_key_value $ret.Headers['Set-Cookie'] '_redmine_session='  
  
# ログインページのcsrf-token取得  
$html = $ret.ParsedHtml  
$csrf_token = get_attribyte_value $html 'csrf-token' 'content'  
[System.Runtime.Interopservices.Marshal]::ReleaseComObject($html) | Out-Null  
  
# セッション情報作成  
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession  
$cookie = New-Object System.Net.Cookie   
$cookie.Name = "_redmine_session"  
$cookie.Value = $session_id  
$cookie.Domain = $redmine_host  
$session.Cookies.Add($cookie);  
  
##################################  
# ログイン処理。  
# ログイン後はチケット登録画面へ  
##################################  
$login_data = @{  
  authenticity_token = $csrf_token;  
  back_url = "http://$redmine_host/projects/$redmine_project/issues/new";  
  username = $username;  
  password = $password;  
}  
$ret = Invoke-WebRequest  "http://$redmine_host/login" -Method POST -WebSession $session -Body $login_data -ContentType "application/x-www-form-urlencoded"  
  
# csrf-token取得  
$html = $ret.ParsedHtml  
$csrf_token = get_attribyte_value $html 'csrf-token' 'content'  
[System.Runtime.Interopservices.Marshal]::ReleaseComObject($html) | Out-Null  
  
##################################  
# チケット登録  
##################################  
Write-Host "チケット登録........................................................"  
$title = Get-Date -format "yyyyMMddHHmmss"  
$ticket_data = @{  
  'utf8' = '✓';  
  authenticity_token = $csrf_token;  
  'issue[is_private]' = 0;  
  'issue[tracker_id]' = 1;  
  'issue[subject]' = "自動登録 $title";  
  'issue[description]' = "わっふるわっふる";  
  'issue[status_id]' = 1;  
  'was_default_status' = 1;  
  'issue[priority_id]' = 2;  
  'issue[start_date]' =  '2019-10-10';  
  'issue[due_date]' =  '';  
  'issue[done_ratio]' = 0;  
  'commit' = '作成'  
}  
$ret = Invoke-WebRequest  "http://$redmine_host/projects/$redmine_project/issues" -Method POST -WebSession $session -Body $ticket_data -ContentType "multipart/form-data"  
$html = $ret.ParsedHtml  
Write-Host $html.title  
[System.Runtime.Interopservices.Marshal]::ReleaseComObject($html) | Out-Null  
  
```

### VBAまたはVBSの場合
MSXML2.XMLHTTPを使用した場合、レスポンスデータのSet-Cookieは削除されてしまいます。（[WireShark](https://www.wireshark.org/)などのパケットアナライザーでみるとデータ自体はきているがライブラリで消している）

 - [VBA Microsoft.XMLHTTP setRequestHeader not sending cookie](https://stackoverflow.com/questions/38306825/vba-microsoft-xmlhttp-setrequestheader-not-sending-cookie?rq=1)

Windows Httpを使用することになるのですが、リダイレクトを含むページがある場合に自動でリダイレクトをさせると_redmine_sessionのやりとりがうまくいかないようです。このことを考慮して実装したコードが以下のようになります。
以下はVBSのサンプルになっていますがWScript.EchoをDebug.Print等に置き換えることでVBAでも流用できると思います。


```vb  
'---------------------------------------------------------------------------  
' Redmineの設定  
'---------------------------------------------------------------------------  
Const redmineHost = "192.168.0.200"  
Const redminePrj = "test1"  
Const user = "user"  
Const password = "pass"  
  
'  
' エスケープ処理  
'https://qiita.com/mima_ita/items/8fc5fab7259835e4bcdd  
'http://www.xtremevbtalk.com/showthread.php?t=152882  
Public Function escape(ByVal StringToEncode )  
    Dim i   
    Dim acode   
    Dim char   
    escape = StringToEncode  
  
    For i = Len(escape) To 1 Step -1  
        acode = AscW(Mid(escape, i, 1))  
        Select Case acode  
            'VBAだと動くがVBSだと動かない。エンコードしても問題ないのでよしとする  
            'Case 48 To 57, 65 To 90, 97 To 122  
            '    ' don't touch alphanumeric chars  
            '  
            Case 32  
                ' replace space  
                escape = Left(escape, i - 1) & "%20" & Mid(escape, i + 1)  
  
            Case Else  
                ' replace punctuation chars with "%hex"  
                char = Hex(acode)  
                If Len(char) > 2 Then  
                    If Len(char) = 3 Then  
                        char = "0" & char  
                    End If  
                    escape = Left(escape, i - 1) & "%u" & char & Mid(escape, i + 1)  
                Else  
                    If Len(char) = 1 Then  
                        char = "0" & char  
                    End If  
                    escape = Left(escape, i - 1) & "%" & char & Mid(escape, i + 1)  
                End If  
        End Select  
    Next  
End Function  
  
' サーバから取得したCookieの値からキーを指定して値を取得する  
Function GetKeyValue(ByVal v, ByVal key)  
  Dim tmp  
  tmp = Mid(v, InStr(v, key) + Len(key))  
  GetKeyValue = Mid(tmp, 1, InStr(tmp, ";") - 1)  
End Function  
  
' 特定の特定の名前を持つ要素の属性を取得  
Function GetAttributeValue(ByVal objHtml, ByVal elemName, Byval attrName)  
  Dim elems  
  Set elems = objHtml.getElementsByName(elemName)  
  GetAttributeValue = elems.Item(0).GetAttribute(attrName)  
End Function  
  
  
  
Dim httpReq  
Set httpReq = CreateObject("WinHttp.WinHttpRequest.5.1")  
httpReq.Option(6) = True ' WinHttpRequestOption_EnableRedirects リダイレクトを無効にする  
  
'-------------------------------------------------------------  
' ログインページに遷移  
'-------------------------------------------------------------  
Call httpReq.Open("GET", "http://" & redmineHost & "/login", False)  
Call httpReq.Send  
  
' セッションID取得  
Dim sessionId  
sessionId = GetKeyValue(httpReq.getResponseHeader("Set-Cookie"), "_redmine_session=")  
  
Dim objHtml  
Set objHtml = CreateObject("htmlfile")  
call objHtml.write(httpReq.responseText)  
  
Dim csrfToken  
csrfToken = GetAttributeValue(objHtml, "csrf-token" ,"content")  
objHtml.Close  
  
WScript.Echo "ログインページ取得-----------------------------"  
WScript.Echo httpReq.Status  
WScript.Echo "_redmine_session：" & sessionId  
WScript.Echo "csrfToken：" & csrfToken  
  
  
'-------------------------------------------------------------  
' ログイン処理  
'-------------------------------------------------------------  
Call httpReq.Open("POST", "http://" & redmineHost & "/login", False)  
Call httpReq.setRequestHeader("Content-Type", "application/x-www-form-urlencoded")  
Call httpReq.setRequestHeader("Cookie", "_redmine_session=" & sessionId)  
Call httpReq.setRequestHeader("Host", redmineHost)  
  
Dim loginData  
loginData = "authenticity_token=" & escape(csrfToken) & _  
            "&back_url=" & "http://" & redmineHost & "/projects/" & redminePrj & "/issues/new" & _  
            "&username=" & user & _  
            "&password=" & password  
Call httpReq.Send(loginData)  
  
WScript.Echo "ログイン実行-----------------------------"  
WScript.Echo httpReq.Status  
sessionId = GetKeyValue(httpReq.getResponseHeader("Set-Cookie"), "_redmine_session=")  
WScript.Echo "_redmine_session：" & sessionId  
  
Dim redirectUrl  
redirectUrl = ""  
If httpReq.Status = 302 Then  
  redirectUrl = httpReq.getResponseHeader("Location")  
End If  
  
If redirectUrl <> "" Then  
  '-------------------------------------------------------------  
  ' チケット登録ページへリダイレクト  
  '-------------------------------------------------------------  
  WScript.Echo "リダイレクト:" & redirectUrl   
  Call httpReq.Open("GET", redirectUrl, False)  
  Call httpReq.Send()  
  WScript.Echo httpReq.Status  
End If  
  
' トークンの再取得  
call objHtml.write(httpReq.responseText)  
csrfToken = GetAttributeValue(objHtml, "csrf-token" ,"content")  
objHtml.Close  
  
  
'-------------------------------------------------------------  
' チケット追加  
'-------------------------------------------------------------  
WScript.Echo "チケット登録-----------------------------"  
sessionId = GetKeyValue(httpReq.getResponseHeader("Set-Cookie"), "_redmine_session=")  
WScript.Echo "_redmine_session：" & sessionId  
WScript.Echo "csrfToken：" & csrfToken  
  
Call httpReq.Open("POST", "http://" & redmineHost & "/projects/" & redminePrj & "/issues", False)  
Call httpReq.setRequestHeader("Content-Type", "multipart/form-data")  
Call httpReq.setRequestHeader("Cookie", "_redmine_session=" & sessionId)  
  
Dim ticketData  
ticketData = "authenticity_token=" & escape(csrfToken) & _  
             "&commit=作成" &_  
             "&issue[is_private]=0" &_  
             "&issue[tracker_id]=1" & _  
             "&issue[subject]=自動登録VBS" & Now() & _  
             "&issue[description]=わっふるわっふる" & _  
             "&issue[status_id]=1" & _  
             "&was_default_status=1" & _  
             "&issue[priority_id]= 2" & _  
             "&issue[start_date]=2019-10-10" &_  
             "&issue[done_ratio]=0"  
Call httpReq.Send(ticketData)  
WScript.Echo httpReq.Status  
If httpReq.Status = 302 Then  
  redirectUrl = httpReq.getResponseHeader("Location")  
End If  
  
If redirectUrl <> "" Then  
  '-------------------------------------------------------------  
  ' チケット登録後のページへリダイレクト  
  '-------------------------------------------------------------  
  WScript.Echo "リダイレクト:" & redirectUrl   
  Call httpReq.Open("GET", redirectUrl, False)  
  Call httpReq.Send()  
  WScript.Echo httpReq.Status  
End If  
  
call objHtml.write(httpReq.responseText)  
WScript.Echo objHtml.title  
objHtml.Close  
  
```

### Pythonの例
HTMLの解析がしんどいので[Beautiful Soup](https://www.crummy.com/software/BeautifulSoup/bs4/doc)を使用した方がいいでしょう。

```python  
import requests  
from bs4 import BeautifulSoup  
import datetime  
  
##############################################  
# redmineの情報  
##############################################  
redmine_host = "192.168.0.200"  # サーバ名  
redmine_project = "test1"       # プロジェクト名  
username = "user"  
password = "password"  
  
# セッションの作成  
session = requests.session()  
  
# ログインページの取得  
ret = session.get('http://' + redmine_host + '/login')  
print(ret.status_code, ret.reason)  
ret.raise_for_status()  
  
session_id = ret.cookies['_redmine_session']  
  
soup = BeautifulSoup(ret.text, 'html.parser')  
elem_token = soup.find('meta', {'name': 'csrf-token'})  
csrf_token = elem_token['content']  
  
# ログイン処理  
cookies = {  
  redmine_host : session_id  
}  
login_data = {  
  'authenticity_token' : csrf_token,  
  'back_url' : "http://' + redmine_host + '/projects/' + redmine_project + '/issues/new",  
  'username' : username,  
  'password' : password  
}  
ret = session.post('http://' + redmine_host + '/login', data=login_data, cookies=cookies)  
print(ret.status_code, ret.reason)  
ret.raise_for_status()  
  
# チケット登録  
print('チケット登録................................')  
soup = BeautifulSoup(ret.text, 'html.parser')  
elem_token = soup.find('meta', {'name': 'csrf-token'})  
csrf_token = elem_token['content']  
  
ticket_data = {  
  'utf8' : '✓',  
  'authenticity_token' : csrf_token,  
  'issue[is_private]' : 0,  
  'issue[tracker_id]' : 1,  
  'issue[subject]' : "自動登録 " + str(datetime.datetime.now()),  
  'issue[description]' : "わっふるわっふる",  
  'issue[status_id]' : 1,  
  'was_default_status' : 1,  
  'issue[priority_id]' : 2,  
  'issue[start_date]' :  '2019-10-10',  
  'issue[due_date]' : '',  
  'issue[done_ratio]' : 0,  
  'commit' : '作成'  
}  
ret = session.post('http://' + redmine_host + '/projects/' + redmine_project + '/issues', data=ticket_data, cookies=cookies)  
print(ret.status_code, ret.reason)  
ret.raise_for_status()  
  
soup = BeautifulSoup(ret.text, 'html.parser')  
print(soup.title)  
```

### ブラウザから送信しているデータを真似する方法のまとめ
ブラウザを介さないのでブラウザより簡単にかつ高速で自動操作が行えます。
同時にそれは、デメリットになる場合があります。
たとえば、JavaScriptでクライアントサイドで動的にページを作成している場合、その処理は動作しません。つまり、ブラウザで操作したときと同様のDOMの構成が返ってくるとは限りません。

この手法を結合試験やシステム試験で使用する場合は、注意してください。
仮にテストデータの入力に使う場合であっても、システム上本来作成できないデータが作成されてしまう場合があるからです。試験の観点に合わせて慎重に導入してください。

また、ブラウザの自動操作と同様にWebアプリケーションの変更によって今まで動いていた自動化スクリプトが動作しなくなるリスクはあるので注意してください。

# Webアプリケーションが提供しているAPIを利用する方法
もっともリスクの少ないWebアプリケーションの自動化の方法です。
ただしWebアプリケーションがAPIを提供しているかどうかは個別の仕様次第になります。

## Redmineのチケット登録の例
これまでにRedmineでチケット登録を行うサンプルをいくつか記述しました、Redmineが提供しているAPIを利用することでシンプルに実装することができます。

まずRedmineの管理画面でRESTAPIを有効にしてください。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/f9df4c72-ef7e-0a5d-3245-b8645b077fb5.png

すると個人設定画面でAPIキーが表示されます。このAPIを使用してRedmineを操作します。
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/aa44ef01-3e1a-41c4-826f-5365812d472e.png


### PowerShellの例
チケット用のXMLを作成してPOSTするだけです。
この際、APIキーをヘッダに付与して送信します。

```powershell  
##################################  
# Redmineの設定値  
##################################  
$redmine_host = "192.168.0.200"  # サーバ名  
$apikey = "60076966cebf71506ae3f2391da649235a2b1d46"  
  
$title = Get-Date -format "yyyyMMddHHmmss"  
$xml = @"  
<issue>  
    <project_id>1</project_id>  
    <subject>RESTAPIテスト $title</subject>  
    <description>試験</description>  
</issue>  
"@  
# セッション情報作成  
$headers = @{  
  'X-Redmine-API-Key' = $apikey;  
  'Content-Type' = 'text/xml';  
}  
  
# 文字化けして登録されるなら以下をいれる  
$sendData = [System.Text.Encoding]::UTF8.GetBytes($xml)  
  
$ret = Invoke-WebRequest http://$redmine_host/issues.xml -Headers $headers -Method POST -WebSession $session -Body $sendData  
Write-Host $ret.Content  
```

### VBAまたVBSの場合
以下はVBSのサンプルになっていますがWScript.EchoをDebug.Print等に置き換えることでVBAでも流用できると思います。

```vb  
'---------------------------------------------------------------------------  
' Redmineの設定  
'---------------------------------------------------------------------------  
Const redmineHost = "192.168.0.200"  
Const apikey = "60076966cebf71506ae3f2391da649235a2b1d46"  
  
  
Dim httpReq  
Set httpReq = CreateObject("WinHttp.WinHttpRequest.5.1")  
  
Call httpReq.Open("POST", "http://" & redmineHost & "/issues.xml", False)  
Call httpReq.setRequestHeader("Content-Type", "text/xml")  
Call httpReq.setRequestHeader("X-Redmine-API-Key", apikey)  
  
Dim xml  
xml = "<issue><project_id>1</project_id> <subject>RESTAPIテスト VBS</subject><description>試験</description></issue>"  
  
Call httpReq.Send(xml)  
WScript.Echo httpReq.Status  
WScript.Echo httpReq.responseText  
  
```

### Pythonの場合

```python  
import requests  
import datetime  
  
xml = """  
<issue>  
    <project_id>1</project_id>  
    <subject>RESTAPIテスト python {0}</subject>  
    <description>試験</description>  
</issue>  
""".format(str(datetime.datetime.now()))  
  
headers = {  
  'X-Redmine-API-Key' : '60076966cebf71506ae3f2391da649235a2b1d46',  
  'Content-Type' : 'text/xml'  
}  
  
r = requests.post('http://192.168.0.200/issues.xml', data=xml.encode('utf-8'), headers=headers)  
print(r.status_code, r.reason)  
print(r.text)  
  
```


なお、Pythonでやるなら[Python Redmine](https://qiita.com/mima_ita/items/1a939db423d8ee295c85)あたりのライブラリを使用したほうが楽だと思います。

# 共通的な注意事項
ここまででWebアプリケーションの自動化の方法についていくつか方法を説明しました。
最後に共通的な注意事項を述べておきたいと思います。

## できることと、やっていいことは違う
おそらくここまでで、多くのWebアプリケーションを自動で操作することが可能になったと思います。
しかしながら、できることと、やっていいことは違うということを常に心がけてください。

API経由以外の自動操作はWebアプリケーション側が想定していない操作になる可能性があります。つまり、いつ動かなくなってもおかしくありませんし、仮に動くからといってやっていい操作とは限りません。
場合によっては規約違反に問われることになります。たとえば広告ブロックして云々とか、複数人で遊ぶブラウザゲームの自動化は、かなりの確率で規約違反になります。
Webアプリケーションを自動化する際は必ず規約を確認してから行うようにしましょう。

また、そういった規約が明記されておらず、不正に当たらないと考えられる場合であっても自動操作はWebアプリケーション側に想定外の負荷を与えることがあります。たとえば、2010年には情報取得目的に図書館の蔵書検索システムに高頻度(1秒に1アクセス程度)のリクエストを送信して偽計業務妨害容疑で逮捕された[岡崎市立中央図書館事件](https://ja.wikipedia.org/wiki/%E5%B2%A1%E5%B4%8E%E5%B8%82%E7%AB%8B%E4%B8%AD%E5%A4%AE%E5%9B%B3%E6%9B%B8%E9%A4%A8%E4%BA%8B%E4%BB%B6)があったことは心に留めておくべきでしょう。

特に社内システムの場合、品質が悪い傾向があるので、根回しをしつつやっておくか、すぐに停止てきる状況かで実施し始めた方が無難です。

## 武器や流派にこだわるな
「武器や流派にこだわるな」という格言は、およそ20年前の名著「[アジャイルソフトウェア開発 (The Agile Software Development Series) ](https://www.amazon.co.jp/dp/4894715791)」の「付録B3 武蔵」の項目にでてきた格言です。

今回、色々な自動化の方法を紹介はしましたが、それは自動化を行うための選択肢を増やして「こだわりを捨ててもらう」意図がありました。

RPAツールは素晴らしく自動化の手助けになります。しかしながら、あきらかに別の方法でやったほうが楽な場合でもRPAツールにこだわるケースがよくみられます。例えばブラウザ画面を介しての自動操作に慣れ親しんだ人はcurlコマンドで済むようなことまで慣れ親しんだという理由だけで困難な技法を選択してしまうケースをよく見かけます。
逆にcurlコマンドでは行うのが困難なことを、それだけでやろうとするケースも同じくらいよく見ます。

普段は使用しない技法であっても必要があるなら採用すべきですし、逆に最も自分が慣れ親しんだ技法であっても状況にそぐわなければ捨てるべきです。

## 自動化スクリプトの管理方法を考えよう
１度動かせばすむスクリプトなのか、定期的に動かすスクリプトなのかによって、スクリプトの管理方法が変わります。
定期的に動かすスクリプトの場合、常にWebアプリケーションのバージョンアップで動作しなくなるというリスクがあります。
このリスクをどう扱うか考えましょう。

たとえば、実際やってエラーとなった時点で修正する時間的余裕のあるものであれば、その時に考えればいいでしょう。
しかし、そういう時間的余裕が確保できないようなスクリプトの場合は、事前にそれを検出する必要があります。
定期的にスモークテストを行う計画を立てるか、Webアプリケーションのリリースノートをチェックする工数をとるか、いずれにせよなんらかの対策が必要になります。

あとは、自動化スクリプトの意図を複数の人間が理解して、メンテナンスできる体制を作るよう必要があります。人間は割と簡単にいなくなります。一誰も意図が分からない自動化スクリプトが動き続ける状態にならないように気を付けましょう。

自動化のコストを見積もる場合、これらの作ったあとのメンテナンスのコストについて忘れずに考えておきましょう。

## 自動化を目的にするのはやめよう
慣れてくると、なんらかの方法で多くのことが自動化できるようになりますが、それを目的とするのはやめましょう。
重大な障害対応を放置して、優先度の低い自動化スクリプトを書いても意味はありません。
全体の状況をみて、効果のありそうなところを自動化しましょう。

## 無理ならあきらめよう
どうしても自動化できないこともあります。
素直にあきらめて別の事をしましょう。


# 参考
 - [Internet Explorer C++ Reference](https://docs.microsoft.com/ja-jp/previous-versions/windows/internet-explorer/ie-developer/platform-apis/hh801968(v=vs.85))
 - [powershellのIEを操作するスクリプトの中のgetElementByIdを使う箇所でエラーが起こる](https://social.technet.microsoft.com/Forums/ja-JP/02cd48a5-5236-4a4e-8a66-0c8a5b67ceaf/powershell12398ie12434258052031612377124271247312463125221250312488?forum=powershellja)
 - [PowerShell Exception 0x800A01B6 while using getElementsByTagName, getElementsByName or getElementByID](https://paullimblog.wordpress.com/2017/07/28/powershell-exception-0x800a01b6-while-using-getelementsbytagname-getelementsbyname-or-getelementbyid/)
 - [Exception from HRESULT: 0x800A138A At line......](https://social.technet.microsoft.com/Forums/ie/en-US/33be53ea-3afe-49e7-b586-41de95593373/exception-from-hresult-0x800a138a-at-line?forum=winserverpowershell)
 - [Cant create HTML document from Hwnd using C#
](https://stackoverflow.com/questions/20873885/cant-create-html-document-from-hwnd-using-c-sharp)
 - [Selenium実践入門](https://www.amazon.co.jp/dp/B07JHNFB2F)
 - [Webブラウザ自動化ツール「Selenium IDE」の今までとこれから](https://www.valtes.co.jp/qbookplus/509)
 - [The default value of timeouts on selenium webdriver
](https://stackoverflow.com/questions/30114976/the-default-value-of-timeouts-on-selenium-webdriver)
 - https://www.seleniumhq.org/
 - [PowerShell & Selenium: Automate Web Browser Interactions – Part I](https://tech.mavericksevmont.com/blog/powershell-selenium-automate-web-browser-interactions-part-i/)
 - [Nodejsを使ってSeleniumでChromeを動かす](https://qiita.com/tonio0720/items/70c13ad304154d95e4bc)
 - [Selenium Nodejs CHROMEDRIVER path](https://stackoverflow.com/questions/26191142/selenium-nodejs-chromedriver-path)
 - [selenium-webdriver](https://seleniumhq.github.io/selenium/docs/api/javascript/index.html)
 - [Can Selenium interact with an existing browser session?](https://stackoverflow.com/questions/8344776/can-selenium-interact-with-an-existing-browser-session)
 - https://developer.chrome.com/extensions
 - https://developer.mozilla.org/ja/docs/Mozilla/Add-ons/WebExtensions/Your_first_WebExtension
 - [Chrome拡張開発: 拡張からページにJavaScriptを送り込みたい](https://qiita.com/suin/items/5e1aa942e654bce442f7)
 - [UiPath Community](https://www.uipath.com/platform-trial)
 - [Installing the Chrome Extension](https://docs.uipath.com/studio/lang-ja/docs/installing-the-chrome-extension)
 - [Web上のリストボックスで複数選択したい](https://forum.uipath.com/t/web/113531/9)
 - https://kashika.biz/uipath_orchestrator_01/
 - https://curl.haxx.se/docs/manpage.html
 - [Windows 10 に curl がプリインストールされてた](https://moyapro.com/2019/04/18/windows10-curl-pre-install/)
 - [Invoke-WebRequestの文字化け](https://qiita.com/zaki-lknr/items/1ae3258d7b77c5e2a2ba)
 - [http.client](https://docs.python.org/ja/3/library/http.client.html)
 - [requestsパッケージ](https://pypi.org/project/requests/2.7.0/)
 - [Beautiful Soup](https://www.crummy.com/software/BeautifulSoup/bs4/doc)
 - https://bitnami.com/stack/redmine/virtual-machine)
 - [岡崎市立中央図書館事件](https://ja.wikipedia.org/wiki/%E5%B2%A1%E5%B4%8E%E5%B8%82%E7%AB%8B%E4%B8%AD%E5%A4%AE%E5%9B%B3%E6%9B%B8%E9%A4%A8%E4%BA%8B%E4%BB%B6)
 - [アジャイルソフトウェア開発 (The Agile Software Development Series) ](https://www.amazon.co.jp/dp/4894715791)」
 - [[VBA]Excel VBAでHTTP通信](https://outofmem.tumblr.com/post/63052619242/vbaexcel-vba%E3%81%A7http%E9%80%9A%E4%BF%A1)
 - [Parse html file using MSHTML in VBScript](https://stackoverflow.com/questions/9931429/parse-html-file-using-mshtml-in-vbscript)
 - [HTTP Requests in VBA: WinHttp, or MSXML2, or ...?](https://social.msdn.microsoft.com/Forums/sqlserver/en-US/a7445768-4e81-4c90-99ad-8173005f2d0c/http-requests-in-vba-winhttp-or-msxml2-or-?forum=windowsgeneraldevelopmentissues)
 - [Windows HTTP](https://docs.microsoft.com/en-us/windows/win32/winhttp/winhttp-start-page)
 - [VBA Microsoft.XMLHTTP setRequestHeader not sending cookie](https://stackoverflow.com/questions/38306825/vba-microsoft-xmlhttp-setrequestheader-not-sending-cookie?rq=1)
 - [Cookie Handling in WinHTTP](https://docs.microsoft.com/en-us/windows/win32/winhttp/cookie-handling-in-winhttp)
 - [Default Integrity Level and Automation](https://blogs.msdn.microsoft.com/ieinternals/2011/08/03/default-integrity-level-and-automation/)
 - [Accessing COM Objects without ProgID](https://community.idera.com/database-tools/powershell/powertips/b/tips/posts/accessing-com-objects-without-progid)
 - [PowershellでInternetExplorerを操作する](https://qiita.com/flasksrw/items/a1ff5fbbc3b660e01d96)
 - [Can Selenium interact with an existing browser session?](https://stackoverflow.com/questions/8344776/can-selenium-interact-with-an-existing-browser-session)
