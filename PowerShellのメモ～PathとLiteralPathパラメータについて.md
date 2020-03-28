# 目的  
PowerShellの[Get-ChildItem](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-childitem?view=powershell-5.1)や[Copy-Item](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/copy-item?view=powershell-5.1)などのいくつかのコマンドレットはPathとLiteralPathパラメータを持っています。  
  
今回は、この違いと使い所さんについて考えてみます。  
  
# パラメータの説明  
これらはいずれもロケーションを指定できるものですが、ワイルドカードを受け付けることができるかどうかが違います。  
  
 - -Path　ワイルドカードを受け付けることができる。  
 - -LiteralPath　記述されたとおりに使用されます。ワイルドカードとして解釈される文字はありません。パスにエスケープ文字が含まれる場合は、単一引用符で囲みます。  
  
  
# パラメータのワイルドカード  
Pathなどのパラメータではワイルドカード文字を使用して、コマンドで単語パターンを作成できます。  
  
たとえば以下のコマンドでは末尾が「.txt」のファイルを列挙します。  
  
```powershell
Get-ChildItem -Path *.txt
```  
  
コマンドレットのパラメータがワイルドカードを認めるかどうかは各コマンドレットのパラメータの説明で「Accept wildcard characters」の項目をみれば確認できます。  
  
下記は[Get-ChildItem](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-childitem)のPathパラメータのドキュメントになります。  
  
![image.png](/image/6711639d-fe74-ad58-9f64-b77ee48fc7f0.png)  
  
  
では次にPowerShellで使用できるワイルドカードパターンの特殊文字を確認してみます。  
  
|ワイルドカード|説明|例|一致する|一致しない|  
|:-----|:----------------------|:-------------------|:-------|:----------|  
|*|文字列の0個以上の文字に一致する|su*|sub1,sub1|a,sab1|  
|?|任意の1文字に一致する|あい??お.txt|あいうえお.txt,あいabお.txt|あいうお.txt|  
|[*\<char\>*-\<char\>*]|連続する範囲の文字に一致する|a[b-c]c.txt|abc.txt,acc.txt|aac.txt|  
|[*\<char\>*\<char\>*]|文字セットの任意の一文字にセットする|a[いa]b.txt|aab.txt,aいb.txt|acb.txt|  
  
windowsのcmd.exeで使用できるワイルドカードの特殊文字と異なることに注意してください。cmd.exeの場合、ワイルドカードとして使える文字は「*」と「?」でした。そして、この文字はWindowsのファイルとして使えない文字です。  
  
PowerShellのワイルドカードの特殊文字「[」、「]」はファイルとして使える文字になっています。また、Windows環境以外で動作することを考える場合、**全てのワイルドカードの特殊文字がファイル名として存在する可能性があります**。  
これが「LiteralPath」というワイルドカードを解釈しないパラメータが必要となる理由になります。  
  
  
  
もし、指定するワイルドカードパターンにワイルドカード文字として解釈されるべきではないリテラル文字が含まれている場合はエスケープ文字としてバッククォート(\`)を用います。  
  
たとえば以下のようなファイルを検索する例を考えてみましょう。この文字のパターンは「test[」+任意の文字+「].txt」とします。  
  
 - test[1].txt  
 - test[2].txt  
 - test[3].txt  
 - test[4].txt  
  
「[」と「]」のリテラルを含むワイルドカードのパターンのサンプルは以下のいずれかになります。  
  
```powershell
Get-ChildItem  -Path test````[*````].txt
Get-ChildItem  -Path "test````[*````].txt"
Get-ChildItem  -Path 'test``[*``].txt'
```  
  
  
# まとめ  
意識的にワイルドカードを使用しない場合は、LiteralPathを用いたほうが安全です。多くの場合、Pathが既定のパラメータになっているので注意してください。  
  
  
# 参考：  
 - [PowerShell　インアクション](https://www.amazon.co.jp/dp/4797337362)  
  
 - [About Wildcards](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_wildcards)  
  
 - [Supporting Wildcard Characters in Cmdlet Parameters](https://docs.microsoft.com/en-us/powershell/developer/cmdlet/supporting-wildcard-characters-in-cmdlet-parameters)  
  
  
  
# よくわからない話(´・ω・｀)  
  
>「[」と「]」のリテラルを含むワイルドカードのパターンのサンプルは以下のいずれかになります。  
>  
```powershell
Get-ChildItem  -Path test````[*````].txt
Get-ChildItem  -Path "test````[*````].txt"
Get-ChildItem  -Path 'test``[*``].txt'
```  
  
とかいいましたが、実際のところ、この挙動がマイクロソフトのドキュメントのどこで保障されたものかが今一わかりませんでした。  
  
バッククォートでエスケープを行うことは[Supporting Wildcard Characters in Cmdlet Parameters](https://docs.microsoft.com/en-us/powershell/developer/cmdlet/supporting-wildcard-characters-in-cmdlet-parameters#handling-literal-characters-in-wildcard-patterns)で記載されています。問題はそのバッククォートの数の妥当性です。  
  
  
[PowerShell　インアクション](https://www.amazon.co.jp/dp/4797337362)では以下のような解説をしています。  
  
>これは１番目のバッククォートがインタープリタによって削除され、２組目のバッククォートがプロバイダによって削除されるためです（２つ目のバッククォートの削除は、リテラル引用符が含まれたファイル名をあらわすために、エスケープを使用できるようにするためのものです）。パターンを単一引用符で囲むとインタープリタが文字列でエスケープ処理をしなくなるため、必要なバッククォートの数が２つに減ります。  
  
正直なところチンときません。しかも最後に身も蓋もないことを言っています。  
  
>この例は、メタ文字の一部をリテラル文字として扱い、残りの部分でパターン照合を実行するため、きわめて複雑です。これをきちんと理解するには、試行錯誤を繰り返すしかありません。  
  
  
[Manipulating Wildcards](https://www.powershellmagazine.com/2012/10/09/manipulating-wildcards/)をみると、[System.Management.Automation.WildcardPattern]::Unescapeでワイルドカードのアンエスケープをしているっぽいので[コノあたり](https://github.com/PowerShell/PowerShell/blob/500595b66aa8deb2009538b28130934dda60f73e/src/System.Management.Automation/namespaces/LocationGlobber.cs)を解析すれば理解できるかもしれませんが、正直、よくわからんです。  
