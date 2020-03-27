# 目的  
PowerShellを使用したレジストリの操作方法についてまとめます。  
レジストリについては下記を参考にしてください。  
  
https://qiita.com/mima_ita/items/ff656af8e4b56ed17a53  
  
  
  
# 操作例  
以下の例ではHKCUを利用していますが、HKLMのレジストリを更新する場合は管理者権限が必要になります。  
  
また今回のサンプル例はWindows10+PowerShell5.1(64bit)となります。  
  
  
## キーの内容を列挙  
  
```powershell
# 指定のキー以下を調べる
Get-ChildItem -LiteralPath 'HKCU:\TestKey'

# 指定のキー以下を再帰的に調べる
Get-ChildItem -LiteralPath 'HKCU:\TestKey' -Recurse

# ワイルドカードを用いて「Su*」で始まるものを列挙
Get-ChildItem -Path 'HKCU:\TestKey\Su*'

# TestKey以下のキーでTestで開始するもの(大文字小文字区別せず）
Get-ChildItem -Path 'HKCU:\TestKey\' -Include "Test*"  -Recurse

```  
  
[Get-ChildItem](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-childitem?view=powershell-5.1)を用いることでレジストキーの子要素を調べることができます。  
  
-Pathオプションを用いてワイルドカードの指定も可能です。  
-Recurseオプションを用いた場合、再帰的に要素を列挙します。この際、-Includeオプションや-Excludeオプションで取得内容を取捨選択できます。  
ファイルシステムと異なり-Filterオプションによるフィルタリングは行えません。  
  
  
## キーの追加  
  
```powershell
# キーの追加
New-Item 'HKCU:\TestKey\x' 
New-Item 'HKCU:\TestKey\y','HKCU:\TestKey\z' 

# 「HKCU:\TestKey\a」が存在しない場合、エラーになる
New-Item 'HKCU:\TestKey\a\b' 

# Forceをつけることで以下のようになる。
# ・存在しない階層構造の場合は構造を作りながらキーを作成する
# ・すでに存在する場合はエントリーを削除して作りなおす
New-Item 'HKCU:\TestKey\a\b' -Force

```  
  
[New-Item](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/new-item?view=powershell-5.1)を用いることでキーの追加が行えます。  
追加するキーをコンマ区切りで指定することで、一度に複数のキーを追加することが可能です。  
  
-Forceオプションを付けると以下のような挙動になります。  
  
 - 存在していないキーを親として指定した場合は強制的にキーを作成します。  
 - 既に存在しているキーがある場合、強制的にキーを作りなおします。すでに存在したキーを削除してから作りなおします。  
  
## エントリーの追加  
  
```powershell
# エントリー作成
New-ItemProperty -LiteralPath 'HKCU:TestKey\a\b' -Name 's1' -PropertyType 'String' -Value 'test'
New-ItemProperty -LiteralPath 'HKCU:TestKey\a\b' -Name 's2' -PropertyType 'MultiString' -Value ('test', 'bbbb', 'ccc')
New-ItemProperty -LiteralPath 'HKCU:TestKey\a\b' -Name 's3' -PropertyType 'ExpandString' -Value 'Neko;%PATH%'

$b = @()
$b += [byte]0x00
$b += [byte]0xFF
New-ItemProperty -LiteralPath 'HKCU:TestKey\a\b' -Name 'b1' -PropertyType 'Binary' -Value $b

New-ItemProperty -LiteralPath 'HKCU:TestKey\a\b' -Name 'v1' -PropertyType 'DWord' -Value 1
New-ItemProperty -LiteralPath 'HKCU:TestKey\a\b' -Name 'v2' -PropertyType 'QWord' -Value 0xffffffff1

New-ItemProperty -LiteralPath 'HKCU:TestKey\a\b','HKCU:TestKey\a\c' -Name 's99' -PropertyType 'String' -Value 'test'

# ワイルドカードが効く
# HKCU:TestKey\a\b と HKCU:TestKey\a\cに作成される
New-ItemProperty -Path 'HKCU:TestKey\a\*' -Name 'x2' -PropertyType 'QWord' -Value 0xffffffff1
```  
  
[New-ItemProperty](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/new-itemproperty?view=powershell-5.1)を使用することでString,MultiString,ExpandString,Binary,DWord,QWordといった値をもつエントリを作成できます。  
レジストリキーのパスを指定する際に、コンマ区切りで複数を指定することにより、複数のキーに対して同じエントリーを追加できます。  
  
-Pathオプションを使用してワイルドカードを使用した場合、複数のキーにエントリが登録される可能性があります。  
  
作成されたエントリは以下のようになります。  
  
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/4f1e66d9-372c-6eca-249e-89e6d64418c6.png  
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/399b2367-4c82-c47f-02bd-ac9e662cad79.png  
ワイルドカードを用いて追加をしたx2についてはHKCU:TestKey\a\bとHKCU:TestKey\a\cの両方に作成されていることが確認できます。  
  
  
## キーの取得  
  
```powershell
# キーをとる
$item = Get-ItemProperty -LiteralPath 'HKCU:TestKey\a\b'
Write-Host $item.v1

# 複数同時に取得も可能
$items = Get-ItemProperty -Path 'HKCU:TestKey\a\b','HKCU:TestKey\a\c'
Write-Host $item[0].PSPath $items[0].s99
Write-Host $item[1].PSPath $items[1].s99

# ワイルドカードを指定すると複数くる可能性もある
$items = Get-ItemProperty -Path 'HKCU:TestKey\a\*'
Write-Host $item[0].PSPath $items[0].x2
Write-Host $item[1].PSPath $items[1].x2

# 存在チェックは以下のように-ErrorAction指定して取得する
$item = Get-ItemProperty -LiteralPath "HKCU:TestKey\a\xxxx" -ea SilentlyContinue
$item -eq $null

```  
  
[Get-Item](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-item?view=powershell-5.1)を使用してキーを取得できます。  
キーのプロパティとしてエントリ名を指定するとレジストリの値が取得できます。  
  
レジストリキーのパスを指定する際に、複数のキーを指定することで、一度に複数のキーを取得が可能です。  
-Pathオプションを用いてワイルドカードを使用した場合は複数のキーが返却される場合があります。  
  
  
## レジストリの値を取得  
  
```powershell
# キーの値をとる
Get-ItemPropertyValue -LiteralPath 'HKCU:TestKey\a\b' -Name s3
Get-ItemPropertyValue -LiteralPath 'HKCU:TestKey\a\b' -Name v1,v2
Get-ItemPropertyValue -LiteralPath 'HKCU:TestKey\a\b','HKCU:TestKey\a\c' -Name s99

# これもワイルドカードを指定すると複数取得される
Get-ItemPropertyValue -Path 'HKCU:TestKey\a\*' -Name x2
```  
  
[Get-ItemProperty](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-itemproperty?view=powershell-5.1)を使用してレジストリの値を取得できます。  
-Nameオプションに複数のエントリ名を指定すると同時に複数の値を取得できます。  
また、-Pathまたは-LiteralPathオプションに複数のレジストリキーへのパスを指定して同時に複数の値を取得することもできます。  
  
-Pathオプションを用いてワイルドカードを使用した場合は複数の値が返却される場合があります。  
  
なお、レジストリの値の型がExpandStringの場合は、展開されて取得されます。  
たとえば「Neko;%PAHT%」という値が入っている場合、以下のように環境変数PATHが展開された文字が返却されます。  
  
```
Neko;C:\Perl64\site\bin;C:\Perl64\bin;略
```  
  
## キーの存在チェックとエントリの存在チェック  
  
```powershell
# 存在するキーの場合は$Trueとなる
Test-Path -LiteralPath "HKLM:\Software\Microsoft\PowerShell\1\ShellIds\Microsoft.PowerShell"

# 存在しないキーの場合は$Falseとなる
Test-Path -LiteralPath "HKLM:\Software\MicrosoftXXX\PowerShell\1\ShellIds\Microsoft.PowerShell"

# 存在するキーの場合は$Trueとなる
$ret = Test-Path -LiteralPath "HKLM:\Software\Microsoft\PowerShell\1\ShellIds\Microsoft.PowerShell","HKLM:\Software\Microsoft\PowerShell\1\ShellIds\ScriptedDiagnostics"
Write-Host $ret[0]
Write-Host $ret[1]

# ワイルドカードも使用可能
Test-Path -Path "HKLM:\Software\Microsoft\PowerShell\1\ShellIds\*.PowerShell"

# エントリーをふくめてはいけない。以下はエントリが存在してもFalseになる
Test-Path -Path "HKLM:\Software\Microsoft\PowerShell\1\ShellIds\Microsoft.PowerShell\ExecutionPolicy"

```  
  
**キー**の存在チェックは[Test-Path](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/test-path?view=powershell-5.1)を用いて行えます。  
  
-LiteralPath または-Pathオプションにレジストリキーを指定して、そのキーが存在する場合はTrue、存在しない場合はFalseを返します。  
  
-LiteralPath または-Pathオプションにレジストリキーを複数指定した場合は、結果は配列として複数返却されます。  
  
-Pathオプションにワイルドカードを指定して存在チェックが可能です。  
  
-LiteralPath または-Pathオプションに指定するパスはあくまでキーまでであり、エントリを含めた場合、正しく動作しません。  
  
エントリーの存在チェックを行うには下記のように一度、キーの取得を行うといいと思います。  
  
```powershell
> $item = Get-ItemProperty -Path 'HKLM:\Software\Microsoft\PowerShell\1\ShellIds\Microsoft.PowerShell'
> $item.ExecutionPolicy -ne $null
True
> $item.ExecutionPolicyNai -ne $null
False
```  
  
  
  
## レジストリの値を更新  
  
```powershell
Set-ItemProperty -LiteralPath  'HKCU:TestKey\a\b' -Name 's2' -Value @"
teststests
testsetst
testest
"@

Set-ItemProperty -LiteralPath  'HKCU:TestKey\a\b','HKCU:TestKey\a\c' -Name s99 -Value xxx123

Set-ItemProperty -Path 'HKCU:TestKey\a\*' -Name x2 -Value 123
```  
  
[Set-ItemProperty](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/set-itemproperty?view=powershell-5.1)を使用してレジストリの値を更新できます。  
  
-Pathまたは-LiteralPathオプションに複数のレジストリキーへのパスを指定して同時に複数の値を更新することもできます。  
Get-ItemPropertyと異なり、Nameに複数の値を指定することはできません。  
  
-Pathオプションにワイルドカードを用いた場合、複数の値が更新される可能性があります。  
  
## エントリ名の改名  
  
```powershell
# エントリーの改名
Rename-ItemProperty -LiteralPath 'HKCU:TestKey\a\c' -Name x2 -NewName newX2
```  
  
[Rename-ItemProperty](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/rename-itemproperty?view=powershell-5.1)を使用してエントリ名を改名できます。  
  
-Pathオプションにワイルドカードを用いた場合、複数のエントリ名が更新される可能性があります。  
  
## キー名の改名  
  
```powershell
# キーの改名
Rename-Item -LiteralPath 'HKCU:TestKey\a\c'  -NewName newC
```  
  
[Rename-Item](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/rename-item?view=powershell-5.1)を用いることでキー名を改名できます。  
  
-Pathオプションにワイルドカードを用いた場合、複数のキー名が更新される可能性があります。  
  
## エントリーのコピー  
  
```powershell
Copy-ItemProperty -LiteralPath 'HKCU:TestKey\a\b' -Name v1 -Destination 'HKCU:TestKey\a'
```  
  
[Copy-ItemProperty](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/copy-itemproperty?view=powershell-5.1)を用いてエントリのコピーを行います。  
  
上記の例は「HKCU:TestKey\a\b」キーのv1エントリが「HKCU:TestKey\a」にコピーされます。  
  
## エントリの移動  
  
```powershell
Move-ItemProperty -LiteralPath 'HKCU:TestKey\a\b' -Name v2 -Destination 'HKCU:TestKey\a'

Move-ItemProperty -Path 'HKCU:TestKey\a\*' -Name s99 -Destination 'HKCU:TestKey\a'
```  
  
[Move-ItemProperty](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/move-itemproperty?view=powershell-5.1)を利用してエントリの移動を行います。  
  
-Pathオプションを指定してワイルドカードを使用した場合、複数のエントリが移動元となり削除される場合があります。  
  
## キーのコピー  
  
```powershell
# aのもつサブキーはコピーされない
Copy-Item -LiteralPath 'HKCU:TestKey\a' -Destination 'HKCU:TestKey\Other'

# サブキーもコピーされる
Copy-Item -LiteralPath 'HKCU:TestKey\a' -Destination 'HKCU:TestKey\Other' -Recurse

```  
  
[Copy-Item](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/copy-item?view=powershell-5.1)を用いてキーのコピーを行います。  
-Recurseオプションを付与しない場合は、コピー元が保持する子要素のキーはコピーされません。  
-Recurseオプションを付与した場合、コピー元が保持する子要素のキーもコピーされます。  
  
**コピー前の状態**  
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/249242fb-9a05-ebf5-9656-e5473154680e.png  
  
**-Recurseオプションを付与しない場合：子要素のキーはコピーされない**  
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/4d0b3ee8-78cc-028d-1839-cbbfdd6dfe91.png  
  
**-Recurseを付与した場合：子要素のキーもコピーされる**  
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/6c788b23-e777-3283-3089-16c681d80916.png  
  
  
## キーの移動  
  
```powershell
Move-Item -LiteralPath 'HKCU:TestKey\a\newC'  -Destination 'HKCU:TestKey\Other'
```  
  
[Move-Item](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/move-item?view=powershell-5.1)を用いてキーを移動します。  
移動元の子要素のキー含めて全て移動します。  
  
  
## エントリーの削除  
  
```powershell
# エントリ削除
Remove-ItemProperty -LiteralPath 'HKCU:TestKey\a\b' -Name v2
Remove-ItemProperty -LiteralPath 'HKCU:TestKey\a\b','HKCU:TestKey\Other' -Name avalue
```  
  
[Remove-ItemProperty](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/remove-itemproperty?view=powershell-5.1)を用いてエントリの削除を行います。  
  
Nameオプションに複数のエントリ名を指定して同時に複数のエントリを削除可能です。  
また、PathまたはLiteralPathオプションに複数のレジストリキーへのパスを指定することでも複数のエントリを削除可能です。  
  
-Pathオプションにワイルドカードを指定した場合は複数のエントリが削除される可能性があります。  
  
## キーの削除  
  
```powershell
# 子の要素がある場合は確認メッセージが表示される
Remove-Item -LiteralPath 'HKCU:TestKey\Other'

# 子の要素があっても無条件で削除
Remove-Item -LiteralPath 'HKCU:TestKey\a' -Recurse
```  
  
[Remove-Item](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/remove-item?view=powershell-5.1)を用いてキーの削除を行います。  
  
-Recurseオプションを付与しない場合かつ、削除対象キーに子要素のキーがある場合、下記のような確認メッセージが表示されます。  
  
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/9984c2da-00d1-c7c5-b808-85e688a0abd5.png  
  
-Recurseオプションを付与した場合は確認なしで削除します。  
  
PathまたはLiteralPathオプションに複数のレジストリキーへのパスを指定して同時に複数のキーを削除可能です。  
  
-Pathオプションにワイルドカードを指定したばあいは複数のエントリが削除される可能性があります。  
  
## トランザクション処理  
レジストリ操作はトランザクション処理が可能です  
  
下記の例では、トランザクションを用いてキーとエントリーを追加する例です  
  
```powershell
Start-Transaction
New-Item  'HKCU:\TestKey\xxxx' -UseTransaction
New-ItemProperty  "HKCU:\TestKey\xxxx" -Name "MyKey" -Value 123 -UseTransaction
Undo-Transaction
# Complete-Transaction
```  
  
まず[Start-Transaction](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/start-transaction?view=powershell-5.1)を実行します。  
その後、New-Itemなどのレジストリ操作を実行しますが、この際、-UseTransactionオプションを付与します。  
トランザクション内で-UseTransactionオプションを付与して操作した操作は[Complete-Transaction](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/complete-transaction?view=powershell-5.1)を行うまでレジストリに反映されません。  
  
もし処理を差し戻す場合は[Undo-Transaction](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/undo-transaction?view=powershell-5.1)を使用します。  
  
# 64bitOSで32bitのプロセスから動かした場合  
64bitOSにて32bitプロセスのPowerShellからレジストリの登録処理を行った場合、レジストリエディターで閲覧すると期待した場所に登録されていません。  
  
具体的に以下の例を見てみましょう。  
  
  
32ビットプロセスのPowerShellから下記を実行する  
  
```
New-Item 'HKLM:\SOFTWARE\TestSoft\1' -Force
New-Item 'Registry::HKEY_LOCAL_MACHINE\SOFTWARE\TestSoft\2' -Force
```  
  
64ビットプロセスのPowerShellから下記を実行する  
  
```
New-Item 'HKLM:\SOFTWARE\TestSoft\3' -Force
New-Item 'Registry::HKEY_LOCAL_MACHINE\SOFTWARE\TestSoft\4' -Force
```  
  
プログラム的には同じ「HKEY_LOCAL_MACHINE\SOFTWARE」の配下にキーを作成しています。  
しかし、実際作成される箇所はことなります。  
  
コンピューター\HKEY_LOCAL_MACHINE\SOFTWARE\TestSoft\  
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/5b9c73a8-cbab-428c-b9f9-777abb5879e5.png  
  
  
コンピューター\HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\TestSoft  
  
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/02c49a44-6558-8e85-a4d2-23f929209e4b.png  
  
32ビットプロセスで動かした場合、「HKLM\SOFTWARE\Wow6432Node」の配下に登録されていることがわかります。  
64ビットOSで32ビットプロセスを動作させてレジストリを操作した場合、このようなリダイレクトが発生します。  
  
この挙動に関しては下記を参照してください。  
https://www.atmarkit.co.jp/ait/articles/1502/19/news120.html  
https://docs.microsoft.com/ja-jp/windows/win32/winprog64/registry-redirector  
  
  
# まとめ  
PowerShellによるレジストリの操作についてまとめました。  
特にトランザクション周りは強力だと思いますので、従来のRegeditやコマンドラインから実行するよりは楽だと思います。  
