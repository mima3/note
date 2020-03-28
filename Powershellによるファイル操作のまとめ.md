# 目的  
今回はPowerShellのファイル操作をまとめてみます。  
  
**検証環境**  
  
|OS|PSVersion|  
|:--|:--|  
|Windows10|5.1|  
|CentOs7|7.0.0-preview.3|  
  
※基本的にWindows10＋PowerShell5.1で確認していますが、シンボリックリンク、ハードリンク等の挙動が変わりそうな箇所はCentOS+PowerShell7.0でも確認してます。  
  
**やりたい事リスト**  
  
|やりたい事|Windows10 コマンドプロンプト|CentOS7 シェル|Powershell|  
|:---------------|:------|:------|:-------|  
|[カレントディレクトリの取得](#カレントディレクトリの取得)|echo %CD%|pwd|Get-Location|  
|[カレントディレクトリの設定](#カレントディレクトリの設定)|cd/pushd/popd|cd|Set-Location/Push-Location/Pop-Location|  
|[空ファイルの作成](#空ファイルの作成)|copy nul test.txt|touch|New-Item|  
|[フォルダの作成](#フォルダの作成)|mkdir|mkdir|New-Item -Type Directory/Mkdir(Windowsのみ)|  
|[シンボリックリンクの作成](#シンボリックリンクの作成)|mklink or mklink /d|ln -s|New-Item|  
|[ジャンクションの作成](#ジャンクションの作成)|mklink /j|-|New-Item|  
|[ハードリンクの作成](#ハードリンクの作成)|mklink /h|ln|New-Item|  
|[ファイルの一覧表示](#ファイルの一覧表示)|dir|ls|Get-ChildItem|  
|[ファイルの検索](#ファイルの検索)|where /R c:\dev\ps\file\ *.txt|find|Get-ChildItem|  
|[ファイルの削除](#ファイルの削除)|del|rm|Remove-Item|  
|[フォルダの削除](#フォルダの削除)|rmdir|rmdir/rm -rf|Remove-Item|  
|[ファイルのコピー](#ファイルのコピー)|copy |cp|Copy-Item|  
|[フォルダのコピー](#フォルダのコピー)|xcopy|cp -a|Copy-Item|  
|[ファイル/フォルダの移動](#ファイル/フォルダの移動)|move|mv|Move-Item|  
|[ファイル/フォルダの改名](#ファイル/フォルダの改名)|ren|mv|Rename-Item|  
|[ファイルの内容表示](#ファイルの内容表示)|type|cat|Get-Content|  
|[ファイルの更新](#ファイルの更新)|echo xxxx>test.txt|echo xxx > test.txt|Set-Content/Out-File|  
|[ファイルの追記](#ファイルの追記)|echo xxxx>>test.txt|echo xxx >>test.txt|Add-Content/Out-File -Append|  
|[ファイルの追記の監視](#ファイルの追記の監視)|-|tail -f|Get-Content -Wait|  
|[ファイルを空にする](#ファイルを空にする)|copy nul test.txt|cp /dev/null access_log|Clear-Content|  
|[ファイルの属性変更](#ファイルの属性変更)|attrib|chmod|Set-ItemProperty|  
|[ファイルの所有者変更](#ファイルの所有者変更)|icacls b.txt /setowner Administrators|chown|Set-Acl |  
|[ファイル中の文字検索](#ファイル中の文字検索)|findstr|grep|Select-String|  
|[フォルダのサイズ取得](#フォルダのサイズ取得)|-|-|Get-ChildItem + Measure-Object|  
|[ファイルの存在チェック](#ファイルの存在チェック)|-|-|Test-Item|  
|[パスの結合](#パスの結合)|-|-|Join-Path|  
|[相対パスから絶対パスの変換](#相対パスから絶対パスの変換)|-|-|Resolve-Path|  
  
# カレントディレクトリの取得  
## 従来の方法  
  
**Windows10のコマンドプロンプトの例**  
  
```
echo %CD%
```  
  
**CentOs7のシェルの例**  
  
```
pwd
```  
  
## PowerShellの例  
  
```text
PS C:\dev\ps\file> Get-Location

Path
----
C:\dev\ps\file
```  
  
[Get-Location](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-location?view=powershell-5.1)を使用することで現在作業しているカレントディレクトリを取得できます。  
ファイルシステムで実行した場合、返却される値は[System.Management.Automation.PathInfo](https://docs.microsoft.com/en-us/dotnet/api/system.management.automation.pathinfo?view=powershellsdk-1.1.0)になっています。  
  
  
  
# カレントディレクトリの設定  
## 従来の方法  
  
**Windows10のコマンドプロンプトの例**  
  
```
cd c:\dev\ps\file
```  
  
**CentOs7のシェルの例**  
  
```
cd ~/test
```  
  
## PowerShellの例  
  
### Set-Locationの例  
  
**例１Set-Locationを使用する場合**  
```powershell:例１Set-Locationを使用する場合
Set-Location -Literal c:\dev\
Set-Location -Path c:\d?v
Set-Location c:\d?v
```  
  
[Set-Location](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/set-location?view=powershell-5.1)を使用してカレントディレクトリを変更可能です。  
  
この際、-Pathまたは-Pathと-LiteralPathを省略をした場合は[ワイルドカード](https://github.com/mima3/note/blob/master/PowerShellのメモ～PathとLiteralPathパラメータについて.md#%E3%83%91%E3%83%A9%E3%83%A1%E3%83%BC%E3%82%BF%E3%81%AE%E3%83%AF%E3%82%A4%E3%83%AB%E3%83%89%E3%82%AB%E3%83%BC%E3%83%89)の指定を受け付けることができます。  
もし、これにより複数のパスが帰った場合は以下のエラーが発生します。  
  
```text
> Set-Location -Path c:\d*
Set-Location : パス 'c:\d*' が複数のコンテナーに解決されるため、場所を設定できません。場所を設定できるのは、同時に 1 つ
のコンテナーのみです。
発生場所 行:1 文字:1
+ Set-Location -Path c:\d*
+ ~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidArgument: (:) [Set-Location]、PSArgumentException
    + FullyQualifiedErrorId : Argument,Microsoft.PowerShell.Commands.SetLocationCommand
```  
  
### スタックを利用したカレントディレクトリの設定例  
  
```text
PS C:\dev\ps> Set-Location c:\dev
PS C:\dev> Push-Location -LiteralPath c:\dev\ps
PS C:\dev\ps> Push-Location -Path c:\d?v\ps\file
PS C:\dev\ps\file> Push-Location  c:\d?v\ps\right
PS C:\dev\ps\right> Get-Location -Stack

Path
----
C:\dev\ps\file
C:\dev\ps
C:\dev


PS C:\dev\ps\right> Pop-Location
PS C:\dev\ps\file> Pop-Location
PS C:\dev\ps> Pop-Location
PS C:\dev>
```  
  
[Push-Location](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/push-location?view=powershell-5.1)はカレントディレクトリをスタックの一番上に積んだのちに、-Path,-Literalで指定したパスをカレントディレクトリとします。  
-Pathを指定した場合、[ワイルドカード](https://github.com/mima3/note/blob/master/PowerShellのメモ～PathとLiteralPathパラメータについて.md#%E3%83%91%E3%83%A9%E3%83%A1%E3%83%BC%E3%82%BF%E3%81%AE%E3%83%AF%E3%82%A4%E3%83%AB%E3%83%89%E3%82%AB%E3%83%BC%E3%83%89)による指定が可能になりますが、対象が複数になった場合、エラーとなります。  
-Path,-LiteralPathを省略してパスを指定した場合、-Pathが指定されたものとしてワイルドカードを受け付けます。  
  
[Get-Location](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-location?view=powershell-5.1)の-Stackオプションを付与することで現在のスタックの状況を表示することが可能です。  
  
[Pop-Loacation](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/pop-location?view=powershell-5.1)を使用するとカレントディレクトリは最後にプッシュされた場所に変更します。また、スタックの一番上は削除されます。  
  
スタックを利用することで関数内でカレントディレクトリを変更する処理を行っても呼出し元の処理に影響を与えないようにすることが可能になります。  
  
```powershell
function test_fun() {
    Push-Location c:\dev\
    Write-Host "なんかの処理:"  (Get-Location).Path
    Pop-Location
}

Set-Location c:\
test_fun
Write-Host "カレントディレクトリがc:\であることの確認:"  (Get-Location).Path

```  
  
スタックには名前を付けて積むことも可能です。  
  
```text
C:\> Push-Location -StackName n1 c:\dev
C:\dev> Push-Location -StackName n1 c:\dev\ps
C:\dev\ps> Push-Location -StackName n2 c:\share
C:\share> Get-Location -StackName n1

Path
----
C:\dev
C:\


C:\share> Get-Location -StackName n2

Path
----
C:\dev\ps


C:\share> Pop-Location -StackName n1
C:\dev> Pop-Location -StackName n2
C:\dev\ps> Get-Location -StackName n1

Path
----
C:\
```  
  
# 空ファイルの作成  
## 従来の方法  
  
**Windows10のコマンドプロンプトの例**  
  
```
echo. 2>test.txt
copy nul test.txt
```  
  
**CentOs7のシェルの例**  
  
```
touch test.txt
```  
  
## PowerShellの例  
  
```powershell
New-Item ./abc[1].txt -Type File
New-Item -Path ./abc[2].txt -Type File
New-Item . -Name abc[3].txt -Type File
New-Item ./あいうえおおおおお.txt
New-Item -Path ./xxx[1].txt,./xxx[2].txt -Type File

# ワイルドカードを指定して作成
New-Item -Path ./???/ -Name abc1.txt -Type File
New-Item ./???/ -Name abc2.txt -Type File

# 既存のファイルを上書きする
New-Item ./abc[1].txt -Type File -Force
```  
  
空ファイルを作成する場合は[New-Item](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/new-item?view=powershell-5.1)のTypeオプションを省略して実行するか、TypeオプションにFileを指定して実行します。  
  
パスの指定の方法としては-Pathオプションにファイル名までのパスを入れるか、-Pathオプションに親フォルダのパスを指定して、-Nameオプションにファイル名を指定します。  
  
-Pathは省略可能となっており、その場合、一番目のパラメータがPathとみなされます。  
  
-Pathオプションは「,」区切りでパスを指定できます。  
  
-Pathには[ワイルドカード](https://github.com/mima3/note/blob/master/PowerShellのメモ～PathとLiteralPathパラメータについて.md#%E3%83%91%E3%83%A9%E3%83%A1%E3%83%BC%E3%82%BF%E3%81%AE%E3%83%AF%E3%82%A4%E3%83%AB%E3%83%89%E3%82%AB%E3%83%BC%E3%83%89)を受け付けることが可能です。  
**また、New-ItemコマンドレットにはLiteralPathが存在しません。**  
そのため、以下のような挙動を行っているようです。  
  
 - -Nameオプションを使用する場合はワイルドカードを受け付ける。  
 - -Nameオプションを使用しない場合はワイルドカードを受け付けない  
  
  
新規作成先にすでにファイルが存在する場合、以下のエラーが発生します  
  
```
New-Item : ファイル 'C:\dev\ps\file\abc[1].txt' は既に存在します。
発生場所 行:1 文字:1
+ New-Item ./abc[1].txt -Type File
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : WriteError: (C:\dev\ps\file\abc[1].txt:String) [New-Item], IOException
    + FullyQualifiedErrorId : NewItemIOError,Microsoft.PowerShell.Commands.NewItemCommand
```  
  
もし強制的に上書きをする場合は「-Force」オプションを付与します。  
  
# フォルダの作成  
## 従来の方法  
  
**Windows10のコマンドプロンプトの例**  
  
```
mkdir xyz
```  
  
**CentOs7のシェルの例**  
  
```
mkdir xyz
```  
  
## PowerShellの例  
  
```powershell
New-Item ./xyz -Type Directory
New-Item -Path ./xyz[2] -Type Directory
New-Item -Path . -Name xyz[3] -Type Directory
New-Item ./xyz1[z],./xyz1[x] -Type Directory
New-Item ./xyz/abc -Type Directory
New-Item ./sonzaisinai/1/2/3/abc -Type Directory

# ワイルドカードの指定
New-Item -Path ./??? -Name yyy -Type Directory

# 存在していてもエラーとしない
New-Item ./xyz -Type Directory -Force


# CentOSでは不可
mkdir -Path xyz[4] -Name xxx
```  
  
フォルダを作成する場合は[New-Item](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/new-item?view=powershell-5.1)のTypeオプションにDirectoryを指定して実行します。  
パスに存在しないフォルダが混ざっていても、途中の存在しないフォルダを作成しながらフォルダを作成していきます。  
  
基本的なオプションの使い方は[空ファイルの作成](#空ファイルの作成)で説明したものと同じです。  
  
またWindowsにかぎり、mkdirで同様の処理が行えます。CentOS7ではシェルのmkdirが優先されるため動作しません。  
  
# シンボリックリンクの作成  
Windowsの場合、シンボリックリンクの作成には管理者権限が必要です。  
  
## 従来の方法  
  
**Windows10のコマンドプロンプトの例**  
  
```
mklink slink_a.txt c:\dev\ps\file\link\a.txt
mklink sdir c:\dev\ps\file\link\sub2 /D
```  
  
**CentOs7のシェルの例**  
  
```
ln ../a.txt ./slink_a.txt -s
ln ../sub2 ./slink_dir -s
```  
  
## PowerShellの例  
  
```powershell
New-Item -Value '../a.txt' -Path './slink.txt'  -ItemType SymbolicLink
New-Item -Value '../sub2' -Path './slink_dir'  -ItemType SymbolicLink

# SymbolicLinkの場合LinkTypeはSymbolicLinkとなる
(Get-Item ./slink.txt).LinkType
(Get-Item ./slink_dir).LinkType

# リンク先を表示
(Get-Item ./slink.txt).Target
(Get-Item ./slink_dir).Target

```  
  
-Valueオプションにリンク先を入力し、-ItemTypeオプションにSymbolicLinkを設定します。  
  
指定のファイルがシンボリックリンクであるか確認するには、[Get-Item](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-item?view=powershell-5.1)を使用してSystem.IO.DirectoryInfoまたはSystem.IO.FileInfoのLinkTypeプロパティとTargetプロパティを確認します。  
  
# ジャンクションの作成  
## 従来の方法  
  
**Windows10のコマンドプロンプトの例**  
  
```
mklink /j junction_dir c:\dev\ps\file\link\sub1
```  
  
## PowerShellの例  
  
```powershell
New-Item -Value '../sub1' -Path './junction_dir'  -ItemType Junction

# Junctionの場合LinkTypeはJunctionとなる
(Get-Item ./junction_dir).LinkType

# リンク先を表示
(Get-Item ./junction_dir).Target
```  
  
-Valueオプションにリンク先を入力し、-ItemTypeオプションにJunctionを設定します。  
もし-Valueにファイルを指定した場合、下記のエラーが発生します。  
  
```text
New-Item : この操作にはディレクトリが必要です。項目 'C:\dev\ps\file\link\a.txt' はディレクトリではありません。
発生場所 行:1 文字:1
+ New-Item -Value '../a.txt' -Path './junction_txt'  -ItemType Junction
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidOperation: (C:\dev\ps\file\link\a.txt:String) [New-Item]、InvalidOperationException
    + FullyQualifiedErrorId : ItemNotDirectory,Microsoft.PowerShell.Commands.NewItemCommand
```  
  
CentOS7+PowerShell7で実行した場合、エラーが表示されずにNew-Itemが終了しますが実際にジャンクションは作成されません。  
  
# ハードリンクの作成  
  
## 従来の方法  
  
**Windows10のコマンドプロンプトの例**  
  
```
mklink /H hardlink_a.txt c:\dev\ps\file\link\a.txt
```  
  
**CentOs7のシェルの例**  
  
```
ln ../a.txt ./link_a.txt
```  
  
## PowerShellの例  
  
```powershell
# Windows10 + PowerShell5.1
New-Item -Value '../a.txt' -Path './hard_link_a.txt'  -ItemType HardLink

# CentOS7+PowerShell7の場合-Valueにフルパスを入れないとエラーになった
New-Item -Value /home/username/test/link/a.txt -Path './hard_link_a.txt'  -ItemType HardLink

# HardLinkの場合LinkTypeはHardLinkとなる
(Get-Item ./hard_link_a.txt).LinkType

# Targetでリンク先のパスを表示する。複数ある場合は複数表示。
# ※CentOS7+PowerShell7だと取得できなかった
(Get-Item ./hard_link_a.txt).Target
```  
  
-Valueオプションにリンク先を入力し、-ItemTypeオプションにHardLinkを設定します。  
もし-Valueにフォルダを指定した場合、下記のエラーが発生します。  
  
```
New-Item : この操作にはファイルが必要です。項目 'C:\dev\ps\file\link\sub1' はファイルではありません。
発生場所 行:1 文字:1
+ New-Item -Value '../sub1' -Path './hard_link_test.txt'  -ItemType Har ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidOperation: (C:\dev\ps\file\link\sub1:String) [New-Item]、InvalidOperationException
    + FullyQualifiedErrorId : ItemNotFile,Microsoft.PowerShell.Commands.NewItemCommand
```  
  
  
# ファイルの一覧表示  
## 従来の方法  
  
**Windows10のコマンドプロンプトの例**  
  
```
dir c:\dev\

# サブディレクトリを再帰的に表示する例
dir c:\dev\ /s 
```  
  
**CentOs7のシェルの例**  
  
```
ls ~/test

# サブディレクトリを再帰的に表示する例
ls ~/test -R
```  
  
## PowerShellの例  
  
### 単純な例  
  
```powershell
# カレントディレクトリの一覧
Get-ChildItem

# パスを指定して検索
Get-ChildItem -LiteralPath ./lstestdir
Get-ChildItem -LiteralPath ./lstestdir,./testdir

# ワイルドカードを指定する
Get-ChildItem -Path ./t*,r*
Get-ChildItem ./t*,r*

# サブフォルダを再帰的に呼び出す
Get-ChildItem -LiteralPath ./lstestdir -Recurse
Get-ChildItem -LiteralPath ./lstestdir -r
```  
  
[Get-ChildItem](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-childitem?view=powershell-5.1)を使用することで指定のパス配下のファイルとディレクトリの一覧を取得できます。  
  
Get-ChildItemは[System.IO.FileInfo](https://docs.microsoft.com/en-us/dotnet/api/system.io.fileinfo?view=netframework-4.8)の配列を返します。  
  
結果をそのままコンソールに出力するとWindows10 + PowerShell5.1では以下のような形式で表示されます。  
  
```
Get-ChildItem -Path C:\Test

Directory: C:\Test

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        2/15/2019     08:29                Logs
-a----        2/13/2019     08:55             26 anotherfile.txt
-a----        2/12/2019     15:40         118014 Command.txt
-a----         2/1/2019     08:43            183 CreateTestFile.ps1
-ar---        2/12/2019     14:31             27 ReadOnlyFile.txt
```  
  
Modeの意味合いは以下のようになります。  
  
 - l (link)  
 - d (directory)  
 - a (archive)  
 - r (read-only)  
 - h (hidden)  
 - s (system).  
  
  
パスを省略して実行した場合はカレントディレクトリの一覧を表示します。  
  
パスの指定は-LiteralPathまたは-Pathを指定して行います。この際、「,」で複数のパスを指定することで一覧表示の対象パスを複数指定することが可能です。  
  
-LiteralPathは-Literalまたは-PsPathと記述することも可能のようです。  
  
-Pathを指定した場合は[ワイルドカード](https://github.com/mima3/note/blob/master/PowerShellのメモ～PathとLiteralPathパラメータについて.md#%E3%83%91%E3%83%A9%E3%83%A1%E3%83%BC%E3%82%BF%E3%81%AE%E3%83%AF%E3%82%A4%E3%83%AB%E3%83%89%E3%82%AB%E3%83%BC%E3%83%89)を受け付けます。  
-LiteralPathも-Pathも省略した場合に指定されたパスは-Pathと同様の動作をします。  
  
サブフォルダの内容も取得したい場合は-Recurseを付与します。  
  
デフォルトの検索オプションでは隠しファイルが表示されないことに気をつけてください。  
以下のように-Forceオプションを付与することで隠しファイルも表示されるようになります。  
  
```powershell
Get-ChildItem -Literal ./lstestdir/test  -Force
```  
  
  
### 属性を指定した一覧取得  
PowerShell3.0以降であれば、ファイルの属性を指定して一覧の取得が可能です。  
  
```powershell
# 隠しファイルのみ表示する
Get-ChildItem -Hidden

# システムファイルのみ表示する
Get-ChildItem -System

# 読み取り専用のみ表示
Get-ChildItem -ReadOnly

# ディレクトリのみ表示
Get-ChildItem -Directory

# ファイルのみ表示
Get-ChildItem -File

# ファイルでかつ隠しファイルの場合表示
Get-ChildItem -File -Hidden

# 複雑な組み合わせで表示→(読み取り専用 and ディレクトリ以外) or (ディレクトリ以外 and 隠しファイル)
Get-ChildItem -Attributes ReadOnly+!Directory, !Directory+Hidden

```  
  
-Attributesオプションは[FileAttributes Enum](https://docs.microsoft.com/en-us/dotnet/api/system.io.fileattributes)を下記の演算子で組み合わせて使用します。  
  
 - ! (NOT)  
 - + (AND)  
 - , (OR)  
  
Windows7に初期インストールされているPowerShell2.0ではこれらのオプションは無効なので注意してください。  
  
  
### 表示内容を絞りこむ  
Get-ChildItemには表示内容を絞り込むオプションがいくつかあります。  
  
```powershell
# 再帰の深さを制限する
Get-ChildItem -LiteralPath c:\dev\ps\file\testdir -Depth 1

# フィルタを指定して検索
Get-ChildItem -LiteralPath c:\dev\ps\file -Filter *.txt
Get-ChildItem -LiteralPath c:\dev\ps\file -Filter *.txt
Get-ChildItem -LiteralPath c:\dev\ps\file -Filter ????.txt
Get-ChildItem c:\dev\ps\file ????.txt

# Excludeオプションを指定して検索 tで始まるファイルは除く
Get-ChildItem c:\dev\ps\file -Exclude t*
# Excludeオプションを指定して検索 tで始まるファイルと ps1で終わるファイルを除く
Get-ChildItem c:\dev\ps\file -Exclude t*,*.ps1

# Includeオプションを指定して検索 txtで終わる文字
Get-ChildItem c:\dev\ps\file -Recurse -Include *.txt
# aまたはbで始まる文字
Get-ChildItem c:\dev\ps\file -Recurse -Include a*,b*

```  
  
#### -depthオプション  
-depthオプションは再帰の深さを指定して検索することができます。  
たとえば以下のような構造のフォルダが存在するとします。  
  
```
C:\DEV\PS\FILE\TESTDIR
├─src1
├─src2
└─test
    ├─a
    │  ├─junction_src3
    │  ├─junction_src4
    │  └─junction_src5
    ├─b
    │  ├─junction_src6
    │  └─junction_src7
    ├─junction_src2
    └─symbolic_src1
```  
  
「-depth 1」で表示対象となるフォルダはsrc1,src2,testまでになります。  
「-depth 0」を指定した場合は再帰を行いません。  
なお、-depthオプションはPowerShell2.0では使用できません。  
  
  
#### -Filterオプション  
-Filterオプションは「*」と「!」のワイルドカードを使用してフィルタを行います。  
このワイルドカードはいわゆるPowerShellの[ワイルドカード](https://github.com/mima3/note/blob/master/PowerShellのメモ～PathとLiteralPathパラメータについて.md#%E3%83%91%E3%83%A9%E3%83%A1%E3%83%BC%E3%82%BF%E3%81%AE%E3%83%AF%E3%82%A4%E3%83%AB%E3%83%89%E3%82%AB%E3%83%BC%E3%83%89)とは異なります。  
  
では実際に以下のフォルダで-Filterと-Pathで与えられるワイルドカードのふるまいについて検証してみます。  
  
```カレントディレクトリの状態
C:\DEV\PS\FILE\LSFILTERTEST
    abcd.log
    abcd.txt
    b.log
    b.txt
    bbb.log
    bbb.txt
    c.log
    c.txt
    cc.log
    cc.txt
    d.log
    d.txt
    [bc].log
    [bc].txt
```  
  
実行例  
  
```text
PS C:\dev\ps\file\lsfiltertest> Get-ChildItem -Path ./*.txt


    ディレクトリ: C:\dev\ps\file\lsfiltertest


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----       2019/09/06     15:43              0 abcd.txt
-a----       2019/09/06     15:43              0 b.txt
-a----       2019/09/06     15:43              0 bbb.txt
-a----       2019/09/06     15:43              0 c.txt
-a----       2019/09/06     15:43              0 cc.txt
-a----       2019/09/06     15:39              0 d.txt
-a----       2019/09/06     15:43              0 [bc].txt


PS C:\dev\ps\file\lsfiltertest> Get-ChildItem -Literal ./ -Filter *.txt


    ディレクトリ: C:\dev\ps\file\lsfiltertest


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----       2019/09/06     15:43              0 abcd.txt
-a----       2019/09/06     15:43              0 b.txt
-a----       2019/09/06     15:43              0 bbb.txt
-a----       2019/09/06     15:43              0 c.txt
-a----       2019/09/06     15:43              0 cc.txt
-a----       2019/09/06     15:39              0 d.txt
-a----       2019/09/06     15:43              0 [bc].txt


PS C:\dev\ps\file\lsfiltertest>
PS C:\dev\ps\file\lsfiltertest> Get-ChildItem -Path ./????.txt


    ディレクトリ: C:\dev\ps\file\lsfiltertest


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----       2019/09/06     15:43              0 abcd.txt
-a----       2019/09/06     15:43              0 [bc].txt


PS C:\dev\ps\file\lsfiltertest> Get-ChildItem -Literal ./ -Filter ????.txt


    ディレクトリ: C:\dev\ps\file\lsfiltertest


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----       2019/09/06     15:43              0 abcd.txt
-a----       2019/09/06     15:43              0 b.txt
-a----       2019/09/06     15:43              0 bbb.txt
-a----       2019/09/06     15:43              0 c.txt
-a----       2019/09/06     15:43              0 cc.txt
-a----       2019/09/06     15:39              0 d.txt
-a----       2019/09/06     15:43              0 [bc].txt


PS C:\dev\ps\file\lsfiltertest>
PS C:\dev\ps\file\lsfiltertest> Get-ChildItem -Path ./[bc].txt


    ディレクトリ: C:\dev\ps\file\lsfiltertest


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----       2019/09/06     15:43              0 b.txt
-a----       2019/09/06     15:43              0 c.txt


PS C:\dev\ps\file\lsfiltertest> Get-ChildItem -Literal ./ -Filter [bc].txt


    ディレクトリ: C:\dev\ps\file\lsfiltertest


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----       2019/09/06     15:43              0 [bc].txt


```  
  
この結果から以下のことがわかります。  
  
 - *については挙動に差はない  
 - ?については-Filterについては任意の１文字を表しますが、それは空文字も含みます。この挙動はcmd.exeのdirのワイルドカードと同じ挙動です。  
 - []については-Filterはサポートしません。  
  
また、-Filterオプションは他のパラメーターよりも効率的に動きます。このことに関する、実際の速度の比較についての情報は以下のブログに記載があります。  
  
**Get-ChildItem and the–Include and –Filter parameters**  
https://tfl09.blogspot.com/2012/02/get-childitem-and-theinclude-and-filter.html  
  
#### -Excludeオプション  
-Excludeオプションで指定した文字のパターンを除外します。  
-LiteralPathオプションと組み合わせた場合、期待通り動作しません。  
  
このオプションは[ワイルドカード](https://github.com/mima3/note/blob/master/PowerShellのメモ～PathとLiteralPathパラメータについて.md#%E3%83%91%E3%83%A9%E3%83%A1%E3%83%BC%E3%82%BF%E3%81%AE%E3%83%AF%E3%82%A4%E3%83%AB%E3%83%89%E3%82%AB%E3%83%BC%E3%83%89)を受け付けることが可能です。  
「,」区切りで複数のパターンを指定できますが、この場合は指定したパターン全てを除外します。  
  
#### -Includeオプション  
-Pathオプションで指定されたフォルダから特定のアイテムを検索します。  
-LiteralPathオプションを使用した場合、期待通り動作しません。  
  
このオプションは-Recurseと共に使用します。  
  
このオプションは[ワイルドカード](https://github.com/mima3/note/blob/master/PowerShellのメモ～PathとLiteralPathパラメータについて.md#%E3%83%91%E3%83%A9%E3%83%A1%E3%83%BC%E3%82%BF%E3%81%AE%E3%83%AF%E3%82%A4%E3%83%AB%E3%83%89%E3%82%AB%E3%83%BC%E3%83%89)を受け付けることが可能です。  
「,」区切りで複数のパターンを指定できますが、この場合は指定したパターン全てを受け付けます。  
  
### シンボリックリンク、ジャンクションを含む場合  
シンボリックリンク、ジャンクションを含むフォルダで下記のコマンドを実行したとします。  
  
```powershell
Get-ChildItem . -Recurse
```  
  
この結果は環境によってことなります。  
  
#### Windows10+Powershell5.1の場合  
  
```
   ディレクトリ: C:\dev\ps\file\link\ps


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d----l       2019/09/08      1:44                junction_dir
d----l       2019/09/08      1:59                slink_dir
-a----       2019/09/08      1:04             12 hard_link_a.txt
-a---l       2019/09/08      1:59              0 slink.txt


    ディレクトリ: C:\dev\ps\file\link\ps\junction_dir


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----       2019/09/08      0:54              9 s1.txt
-a----       2019/09/08      0:54              9 s2.txt


    ディレクトリ: C:\dev\ps\file\link\ps\slink_dir


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----       2019/09/08      0:55              9 s2_1.txt
-a----       2019/09/08      0:55              9 s2_2.txt

```  
  
シンボリックリンク、ジャンクション、ハードリンクについて、Modeに「l」が付与されていることが確認できます。  
また、シンボリックリンク、ジャンクションのフォルダを再帰的に探査していきます。  
  
#### CentOS7+Powershell7の場合  
  
```
    Directory: /home/xxxxxxxx/test/link/ps

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
l----          2019/09/08     2:03                slink_dir -> /home/xxxxxxxx/t
                                                  est/link/sub2
-----          2019/09/08     0:58              7 hard_link_a.txt
l----          2019/09/08     1:55              8 slink_a.txt -> /home/xxxxxxxx
                                                  /test/link/a.txt
l----          2019/09/08     2:01              8 slink.txt -> /home/xxxxxxxx/t
                                                  est/link/a.txt
```  
  
シンボリックリンク、ハードリンクについて、Modeに「l」が付与されていることが確認できます。  
そして、シンボリックリンクの再帰的な探査は行われません。  
  
もしシンボリックリンク内も探査したい場合は以下のように-FollowSymlinkをつけます。  
  
```powershell
Get-ChildItem . -Recurse -FollowSymlink
```  
  
  
### 出力書式の変更  
パイプを使って書式整形用のコマンドレットに渡すことで様々な書式で表示することが可能です。  
  
#### Format-Wide  
  
[Format-Wide](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/format-wide?view=powershell-5.1)は「dir /w」のような幅広いテーブルで結果を返します。  
  
```powershell
Get-ChildItem | Format-Wide
```  
  
```text
    ディレクトリ: C:\dev\ps\file\lsfiltertest



abcd.log                                                                 abcd.txt
b.log                                                                    b.txt
bbb.log                                                                  bbb.txt
c.log                                                                    c.txt
cc.log                                                                   cc.txt
d.log                                                                    d.txt
[bc].log                                                                 [bc].txt
```  
  
#### Out-GridView  
[Out-GridView](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/out-gridview?view=powershell-5.1)は結果をGUIで表示します。  
  
```powershell
Get-ChildItem | Out-GridView
```  
  
![image.png](/image/03bcd0b2-71e3-cf98-6802-40d8209aee28.png)  
  
LinuxやMacなどのWindows以外のプラットフォームにおいて標準では使用できません。  
しかし、Microsoft.PowerShell.GraphicalToolsモジュールをインストールすると使用できるようです（未検証）  
  
**[PowerShell] 帰ってきたOut-GridView**  
https://dev.classmethod.jp/server-side/out-gridview-returns-powershell-core/  
  
  
# ファイルの検索  
## 従来の方法  
  
**Windows10のコマンドプロンプトの例**  
  
```
where /R c:\dev\ps\file\ *.txt
```  
  
**CentOs7のシェルの例**  
  
```
find ~/test *.txt
```  
  
## PowerShellの例  
```powershell
Get-ChildItem -LiteralPath c:\dev\ps\file\lstestdir -Filter *.txt -Recurse -Force

Get-ChildItem -Path c:\dev\ps\file\lstestdir -Include *.txt -Recurse -Force

# 100バイトより大きいtxtファイルを取得
Get-ChildItem . -Filter *.txt -r | Where-Object { $_.Length -gt 100 }
```  
  
[表示内容を絞りこむ](#表示内容を絞りこむ)で紹介したようにGet-ChildItemの-Filterまたは-Inputオプションを用います。  
  
オプションだけで対応できない場合はWhere-Objectを用いてプロパティを使用して条件を抽出します。  
  
  
# ファイルの削除  
## 従来の方法  
  
**Windows10のコマンドプロンプトの例**  
  
```
del test.txt
```  
  
**CentOs7のシェルの例**  
  
```
rm test.txt
```  
  
## PowerShellの例  
  
```powershell
Remove-Item -LiteralPath test.txt
Remove-Item -LiteralPath test.txt,test2.txt
Remove-Item -Path *.txt
Remove-Item *.txt

# 読み取り専用ファイルを削除する
Remove-Item -LiteralPath readonly.txt -Force

# 何が削除されるか事前に確認する
Remove-Item -Path *.txt -WhatIf

# 削除の確認メッセージを表示する
Remove-Item -Path *.txt -Confirm

# カレントディレクトリのtxt拡張子のファイルをすべて消す
Remove-Item -Path * -Filter *.txt -WhatIf
Remove-Item -Path * -Include *.txt -WhatIf

# カレントディレクトリ以下の全てのフォルダのtxt拡張子ファイルを削除する
Get-ChildItem . -Filter *.txt -Recurse | Remove-Item -WhatIf
```  
  
ファイルの削除は[Remove-Item](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/remove-item?view=powershell-5.1)で行います。  
  
-Pathオプションを指定した場合、[ワイルドカード](https://github.com/mima3/note/blob/master/PowerShellのメモ～PathとLiteralPathパラメータについて.md#%E3%83%91%E3%83%A9%E3%83%A1%E3%83%BC%E3%82%BF%E3%81%AE%E3%83%AF%E3%82%A4%E3%83%AB%E3%83%89%E3%82%AB%E3%83%BC%E3%83%89)による指定が可能になります。  
-LiteralPathと-Pathオプションを省略した場合、-Pathと同じ動作をします。  
  
-LiteralPathと-Pathオプションには削除対象のパスをコンマ区切りで複数指定可能です。  
  
読み取り専用のファイルを削除する場合、以下のようなエラーとなります。  
![image.png](/image/ef819f0a-3c6f-cca8-5a4b-7e2e299fecf8.png)  
この場合は-Forceオプションを付与することで削除がおこなえます。  
  
-WhatIfオプションを付与して実行することで、削除を行わず、削除対象のみを表示します。これはワイルドカードを使用した際に事前に影響を調べるのに有効です。  
![image.png](/image/7c77e468-6a98-ba65-39bd-4015cfe7f301.png)  
  
  
-Confirmオプションを付与して実行することで各ファイルの削除確認を行いながら削除を進めることが可能になります。  
![image.png](/image/4f8c5685-ebdd-e6b9-3364-27be8c9e630a.png)  
  
[ファイルの一覧表示](#ファイルの一覧表示)のGet-ChildItemで出てきた-Filterオプションと-Includeオプションを利用して削除するファイルを絞りこむこともできます。  
  
またGet-ChildItemでファイルを絞り込んだ後にパイプを使ってRemove-Itemに渡すことで削除も可能です。  
この際は、予期せぬファイルが消える可能性があるのでWhat-Ifを使用して事前に削除されるファイルを確認したのち削除した方が安全です。  
  
また、Get-ChildItemでファイルを絞る際、シンボリックリンクやジャンクションを使用している場合は、リンク先も削除される可能性があるので慎重に削除しましょう。  
  
  
# フォルダの削除  
## 従来の方法  
  
**Windows10のコマンドプロンプトの例**  
  
```
rmdir testdir
rmdir /Q /S lsfiltertest
```  
  
**CentOs7のシェルの例**  
  
```
rmdir testdir
rm -rf y
```  
  
## PowerShellの例  
  
```powershell
Remove-Item lsfiltertest
Remove-Item lsfiltertest -Recurse
```  
  
[ファイルの削除](#ファイルの削除)と同じようにRemove-Itemを使用してファルダを削除します。  
  
削除するフォルダにファイルが存在する場合、下記の確認メッセージが表示されます。  
  
![image.png](/image/f99bccc1-b2fc-2c43-4cf7-0c517eb48fed.png)  
  
この確認メッセージを出さないで削除するには-Recurseオプションを利用します。  
  
ジャンクション、シンボリックリンクを含むフォルダが存在する場合は、リンク先を破壊する可能性があります。  
これらを安全に削除する方法は下記を参考にしてください。  
  
**.NETでディレクトリを消すのがこんなに面倒なわけがない**  
https://github.com/mima3/note/blob/master/.NETでディレクトリを消すのがこんなに面倒なわけがない.md  
  
# ファイルのコピー  
## 従来の方法  
  
**Windows10のコマンドプロンプトの例**  
  
```
copy x.txt xxxx
```  
  
**CentOs7のシェルの例**  
  
```
cp x.txt xxxx
```  
  
## PowerShellの例  
  
```powershell
Copy-Item -LiteralPath ./x.txt -Destination xxxx
Copy-Item -LiteralPath a.txt,b.txt -Destination xxxx
Copy-Item -LiteralPath ./x.txt -Destination y.txt
Copy-Item -Path x*.txt -Destination xxxx
Copy-Item y*.txt xxxx

# 読み取り専用がコピー先の場合
Copy-Item b.txt readonly.txt -Force

Copy-Item y*.txt xxxx -WhatIf
Copy-Item y*.txt xxxx -Confirm

# カレントディレクトリのすべてのファイルとフォルダのうちtで始まるものをコピーする
Copy-Item * xxxx -Filter t* -WhatIf

# カレントディレクトリのすべてのファイルとフォルダのうちtで始まるもの以外をコピーする
Copy-Item * xxxx -Exclude t* -WhatIf
```  
  
[Copy-Item](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/copy-item?view=powershell-5.1)を使用してファイルのコピーが可能です。  
  
-LiteralPathまた-Pathオプションにコピー元を指定します。  
  
-Pathオプションを指定した場合、[ワイルドカード](https://github.com/mima3/note/blob/master/PowerShellのメモ～PathとLiteralPathパラメータについて.md#%E3%83%91%E3%83%A9%E3%83%A1%E3%83%BC%E3%82%BF%E3%81%AE%E3%83%AF%E3%82%A4%E3%83%AB%E3%83%89%E3%82%AB%E3%83%BC%E3%83%89)による指定が可能になります。  
-LiteralPathと-Pathオプションを省略した場合、第一引数をコピー元のパスとして取り扱います。この際、-Pathと同じ動作をします。  
  
-LiteralPathと-Pathオプションにはコピー元のパスをコンマ区切りで複数していできます。  
  
-Destinationにはコピー先のフォルダまたは、コピー先のファイル名を指定します。  
  
コピー先のファイルが読み取り専用の場合はエラーとなります。  
この場合、-Forceオプションを付与してコピーします。  
  
-WhatIfオプションを使用することで事前に変更されるパスの確認も可能です。  
![image.png](/image/3ee4b1b9-ea1a-0588-7756-dd3ccadd32c5.png)  
  
-Confirmオプションで確認メッセージが表示されます。  
![image.png](/image/e815a219-b7a6-5ddf-bec0-06fcaaed68aa.png)  
  
[ファイルの一覧表示](#ファイルの一覧表示)のGet-ChildItemで出てきた-Filter,-Include-Excludeオプションを利用して削除するファイルを絞りこむこともできます。  
  
# フォルダのコピー  
## 従来の方法  
  
**Windows10のコマンドプロンプトの例**  
  
```
xcopy /e /s /h ./testdir ./xxxx
```  
  
以下のようになる  
  
```
xxxx
  testdirの子要素1
    孫要素
  testdirの子要素2
```  
  
**CentOs7のシェルの例**  
  
```
cp ./testdir ./xxxx -R
```  
  
以下のようになる  
  
```
xxxx
  testdir
    testdirの子要素1
      孫要素
    testdirの子要素2
```  
  
## PowerShellの例  
  
```powershell
Copy-Item -Literal ./testdir -Destination xxxx -Recurse
```  
  
[ファイルのコピー](#ファイルのコピー)と同様にCopy-Itemを使用してフォルダのコピーを行います。  
コピー先に指定したフォルダの配下にコピー先のディレクトリが作成されます。  
  
この挙動はCentOS7側のフォルダコピー処理と同じです。  
  
  
# ファイル/フォルダの移動  
## 従来の方法  
  
**Windows10のコマンドプロンプトの例**  
  
```
# ファイルの移動
move a.txt b.txt
move b.txt folder

# フォルダの移動
move folder folder2
```  
  
**CentOs7のシェルの例**  
  
```
# ファイルの移動
mv a.txt b.txt
mv b.txt folder

# フォルダの移動
mv folder folder2
```  
  
## PowerShellの例  
  
```powershell
Move-Item -LiteralPath a.txt -Destination b.txt
Move-Item -LiteralPath b.txt -Destination ./folder
Move-Item -LiteralPath ./folder -Destination ./folder2

# ワイルドカードを指定する例
Move-Item -Path a*.txt -Destination ./folder2
Move-Item b*.txt ./folder2

```  
  
ファイルとフォルダの移動は[Move-Item](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/move-item?view=powershell-5.1)を用いて行います。  
  
-LiteralPathまたは-Pathには移動元のパスを指定します。  
-Pathオプションを指定した場合、[ワイルドカード](https://github.com/mima3/note/blob/master/PowerShellのメモ～PathとLiteralPathパラメータについて.md#%E3%83%91%E3%83%A9%E3%83%A1%E3%83%BC%E3%82%BF%E3%81%AE%E3%83%AF%E3%82%A4%E3%83%AB%E3%83%89%E3%82%AB%E3%83%BC%E3%83%89)による指定が可能になります。  
-LiteralPathと-Pathオプションを省略した場合、第一引数を移動元のパスとして取り扱います。この際、-Pathと同じ動作をします。  
  
-Destinationオプションには移動先のパスを指定します。-Destinationオプションを省略した場合は第二引数を移動先のパスとします。  
  
-WhatIfオプションを使用することで事前に変更されるパスの確認も可能です。  
  
-Confirmオプションで確認メッセージが表示されます。  
  
# ファイル/フォルダの改名  
## 従来の方法  
  
**Windows10のコマンドプロンプトの例**  
  
```
ren x.txt xrenamed.txt
```  
  
**CentOs7のシェルの例**  
  
```
mv x.txt xrenamed.txt
```  
  
## PowerShellの例  
  
```powershell
Rename-Item -Path test.txt -NewName test2.txt
Rename-Item test2.txt test3.txt

# カレントディレクトリの全てのファイルの拡張子をtxt->logに変更
Get-ChildItem *.txt | Rename-Item -NewName { $_.name -Replace '\.txt$','.log' }
```  
  
ファイルまたはフォルダの改名には[Rename-Item](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/rename-item?view=powershell-5.1)を使用します。  
  
複数のファイルの改名を同時に行う場合はGet-ChildItemでファイルを取得してパイプを用いてRename-Itemに渡します。  
  
-WhatIfオプションを使用することで事前に変更されるパスの確認も可能です。  
  
-Confirmオプションで確認メッセージが表示されます。  
  
  
# ファイルの内容表示  
## 従来の方法  
  
**Windows10のコマンドプロンプトの例**  
  
```
type x.txt
```  
  
**CentOs7のシェルの例**  
  
```
cat x.txt
```  
  
## PowerShellの例  
  
#### Get-Content  
  
```powershell
Get-Content -Literal ../x.txt
Get-Content -Literal x.txt,y.txt
Get-Content -Path *.txt
Get-Content  *.txt

# 最初の10行取得
Get-Content lines.txt -TotalCount 10

# 最後の10行取得
Get-Content lines.txt -Tail 10

# エンコーディングを指定して表示
Get-Content utf8.txt -Encoding Utf8

# バイト配列の取得
Get-Content -Path C:\temp\test.txt -Encoding Byte -Raw
```  
  
ファイルの内容表示を行うには[Get-Content](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-content?view=powershell-5.1)を使用します。  
  
-LiteralPath,-Pathオプションには表示対象のファイルパスを指定します。  
「,」区切りで複数指定した場合、複数のファイルの内容を表示します。  
  
-Pathオプションを指定した場合、[ワイルドカード](https://github.com/mima3/note/blob/master/PowerShellのメモ～PathとLiteralPathパラメータについて.md#%E3%83%91%E3%83%A9%E3%83%A1%E3%83%BC%E3%82%BF%E3%81%AE%E3%83%AF%E3%82%A4%E3%83%AB%E3%83%89%E3%82%AB%E3%83%BC%E3%83%89)による指定が可能になります。  
  
-LiteralPath,-Pathオプションを省略した場合は、指定したパスを-Pathオプションと同じ方法で開きます。  
  
-TotalCountオプションを指定した場合、先頭から指定した行数のみ表示します。  
-Tailオプションを指定した場合、末尾から指定した行数のみ表示します。  
  
-TotalCount,-Tailオプションは対象のファイルの改行コードに影響せずに行数を取得してくれるようです。  
  
-Encodingを指定して文字コードを指定したファイルの表示や、バイト配列の取得が可能です。  
PowerShell6.0より前は-Encodingはコードページの指定ができず、EUC-JPなどは表示できません。また、UTF8もBOM付きしかサポートしていません。  
省略した場合の挙動は「Default」となり、システムの規定のコードページになります。日本語OSの場合はCP932です。  
  
#### PowerShell5.1+Win10(日本語版） でのEncodingの試験  
各種文字コードのファイルがどのEncodingの指定で表示されるかを検証した結果が以下のようになります。  
  
||Default|Unicode|UTF8 |  
|:-----|:------|:-------|:-------|  
|EUC-JAのファイル|×|×|×|  
|SJISのファイル|〇|×|×|  
|UNICODEのファイル|〇|〇|〇|  
|UTF8（BOMなし）のファイル|×|×|〇|  
|UTF8（BOMあり）のファイル|〇|〇|〇|  
  
### System.IO.Fileを利用する方法  
  
PowerShell5.1以前のGet-Contentはいくつかのエンコーディングしかサポートしていません。  
そのため、EUCなどのファイルを読み込む際は.NETのクラスを直接操作する必要があります。  
  
```powershell
	$enc = [System.Text.Encoding]::GetEncoding(20932)
	[System.IO.File]::ReadAllLines("c:\dev\ps\file\contenttest\euc.txt", $enc)
```  
  
# ファイルの更新  
## 従来の方法  
  
**Windows10のコマンドプロンプトの例**  
  
```
echo xxxx>test.txt
```  
  
**CentOs7のシェルの例**  
  
```
echo xxxx>test.txt
```  
  
## PowerShellの例  
  
### Set-Contentを使用する例  
  
```powershell
Set-Content -LiteralPath .\a.txt -Value 'Hello, World'
Set-Content -LiteralPath a1.txt,a2.txt -Value 'Hello, World'
Set-Content -Path .\b*.txt -Value 'Hello, World'
Set-Content .\c*.txt -Value 'Hello, World'

'Hello, World' | Set-Content -Path .\b*.txt

# UTF8(BOMあり）として更新
Set-Content ./d.txt -Value 'わたしはカモメ' -Encoding Utf8

# バイナリファイルとして更新
Set-Content ./byte.txt -Encoding Byte -Value @([byte]0x30,[byte]0x31,[byte]0x32)

# 事前の確認
Set-Content -Path .\b*.txt -Value 'Hello, World' -WhatIf
```  
  
[Set-Content](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/set-content?view=powershell-5.1)を用いることでファイルに更新が行えます。  
  
-Literal,-Pathオプションには更新対象のファイルパスを指定します。  
「,」区切りで複数指定した場合、複数のファイルを更新します。  
  
-Pathオプションを指定した場合、[ワイルドカード](https://github.com/mima3/note/blob/master/PowerShellのメモ～PathとLiteralPathパラメータについて.md#%E3%83%91%E3%83%A9%E3%83%A1%E3%83%BC%E3%82%BF%E3%81%AE%E3%83%AF%E3%82%A4%E3%83%AB%E3%83%89%E3%82%AB%E3%83%BC%E3%83%89)による指定が可能になります。  
  
ワイルドカードを使用していない場合、指定のファイルが存在しなければ新規にファイルを作成します。  
  
-Valueオプションに書き込む内容を設定します。パイプラインから値を受け付けることもできます。  
  
-Encodingを指定して文字コードを指定したファイルの表示や、バイト配列の取得が可能です。  
PowerShell6.0より前は-Encodingはコードページの指定ができず、EUC-JPなどは表示できません。また、UTF8もBOM付きしかサポートしていません。  
省略した場合の挙動は「Default」となり、システムの規定のコードページになります。日本語OSの場合はCP932です。  
  
読み取り専用ファイルを書きこんだ場合、規定ではエラーとなります。この場合は-Forceオプションを付与することで上書きが可能になります。  
  
-WhatIfオプションを使用することで事前に更新されるファイルの確認も可能です。  
  
-Filter ,-Include, -Exclude オプションで更新対象のファイルを制御できます。  
  
使用中は Read Lock/Write Lock がかかるので長時間実行されているコマンドのログに使用するのは避けたほうがいいです。  
  
### Out-Fileを使用する例  
  
```powershell
Out-File -LiteralPath .\a.txt -InputObject 'The end of the world'
Out-File -FilePath .\b.txt -InputObject 'The end of the world'
Out-File  .\c.txt -InputObject 'The end of the world'

Get-Process | Out-File -FilePath .\Process.txt 

Out-File c.txt -InputObject '私はかもめ' -Encoding Utf8

```  
  
[Out-File](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/out-file?view=powershell-5.1)を使用してファイルを更新可能です。  
  
-LiteralPath,-FilePathオプションには更新対象のファイルパスを指定します。  
-FilePathには[ワイルドカード](https://github.com/mima3/note/blob/master/PowerShellのメモ～PathとLiteralPathパラメータについて.md#%E3%83%91%E3%83%A9%E3%83%A1%E3%83%BC%E3%82%BF%E3%81%AE%E3%83%AF%E3%82%A4%E3%83%AB%E3%83%89%E3%82%AB%E3%83%BC%E3%83%89)による指定が可能ですが、複数更新対象がある場合以下のエラーとなります。  
![image.png](/image/86e7c7e5-244e-c3cd-0813-b60b3d38e418.png)  
また、ドキュメントにも「Accept wildcard characters: False」とあるので指定できてもやめときましょう。  
  
  
-InputObjectはファイルに書き込むオブジェクトを指定します。  
パイプラインから入力を受け付けることも可能です。  
  
-Encodingを指定して文字コードを指定したファイルの表示や、バイト配列の取得が可能です。  
PowerShell6.0より前は-Encodingはコードページの指定ができず、EUC-JPなどは表示できません。また、UTF8もBOM付きしかサポートしていません。  
省略した場合の挙動は「UNICODE」となります。  
  
読み取り専用ファイルを書きこんだ場合、規定ではエラーとなります。この場合は-Forceオプションを付与することで上書きが可能になります。  
  
-WhatIfオプションを使用することで事前に更新されるファイルの確認も可能です。  
  
-Confirmオプションを使用することで書き込み前に確認メッセージが表示されます。  
  
Set-ContentとOut-Fileの比較については下記を参考にしてください。  
  
**PowerShellの Out-File と Set-Content あるいは Out-File -Append と Add-Content の違い**  
https://tech.guitarrapc.com/entry/2014/02/11/061627  
  
**PowerShell Set-Content and Out-File - what is the difference?**  
https://stackoverflow.com/questions/10655788/powershell-set-content-and-out-file-what-is-the-difference  
  
### System.IO.Fileを利用する方法  
  
PowerShell5.1以前のSet-Content/Out-FileのEncodingはいくつかのエンコーディングしかサポートしていません。  
そのため、EUCやUTF8のBOMなしを書き込むさいは.NETのクラスを直接操作する必要があります。  
  
  
```powershell
	$enc = [System.Text.Encoding]::GetEncoding(20932)
	[System.IO.File]::WriteAllLines("c:\dev\ps\file\contenttest\euc_out.txt", "ねこ", $enc)

	$enc = New-Object System.Text.UTF8Encoding($False)
	[System.IO.File]::WriteAllLines("c:\dev\ps\file\contenttest\utf8_cout.txt", "いぬ", $enc)
```  
  
# ファイルの追記  
## 従来の方法  
  
**Windows10のコマンドプロンプトの例**  
  
```
echo xxxx>>test.txt
```  
  
**CentOs7のシェルの例**  
  
```
echo xxx >>test.txt
```  
  
## PowerShellの例  
  
### Add-Contentを使用する例  
  
```powershell
Add-Content -LiteralPath .\a.txt -Value 'end of file'
Add-Content -LiteralPath a1.txt,a2.txt -Value 'end of file'
Add-Content -Path .\b*.txt -Value 'end of file'
Add-Content .\c*.txt -Value 'end of file'

# パイプラインの例。a1の内容をa2に追記
Get-Content a1.txt | Add-Content a2.txt

# UTF8(BOMあり）に追記
Add-Content ./d.txt -Value 'あのこは石ころ' -Encoding Utf8

# バイナリファイルに追記
Add-Content ./byte.txt -Encoding Byte -Value @([byte]0x30,[byte]0x31,[byte]0x32)

```  
  
[Add-Content](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/add-content?view=powershell-5.1)を用いることでファイルに更新が行えます。  
  
-LiteralPath,-Pathオプションには更新対象のファイルパスを指定します。  
「,」区切りで複数指定した場合、複数のファイルを更新します。  
  
-Pathオプションを指定した場合、[ワイルドカード](https://github.com/mima3/note/blob/master/PowerShellのメモ～PathとLiteralPathパラメータについて.md#%E3%83%91%E3%83%A9%E3%83%A1%E3%83%BC%E3%82%BF%E3%81%AE%E3%83%AF%E3%82%A4%E3%83%AB%E3%83%89%E3%82%AB%E3%83%BC%E3%83%89)による指定が可能になります。  
  
ワイルドカードを使用していない場合、指定のファイルが存在しなければ新規にファイルを作成します。  
  
-Valueオプションに書き込む内容を設定します。パイプラインから値を受け付けることもできます。  
  
-Encodingを指定して文字コードを指定したファイルの表示や、バイト配列の取得が可能です。  
PowerShell6.0より前は-Encodingはコードページの指定ができず、EUC-JPなどは表示できません。また、UTF8もBOM付きしかサポートしていません。  
省略した場合の挙動は「Default」となり、システムの規定のコードページになります。日本語OSの場合はCP932です。  
  
読み取り専用ファイルを書きこんだ場合、規定ではエラーとなります。この場合は-Forceオプションを付与することで上書きが可能になります。  
  
-WhatIfオプションを使用することで事前に更新されるファイルの確認も可能です。  
  
-Filter ,-Include, -Exclude オプションで更新対象のファイルを制御できます。  
  
使用中は Read Lock/Write Lock がかかるので長時間実行されているコマンドのログに使用するのは避けたほうがいいです。  
  
### Out-Fileに-Appendオプションを付与して使用する例  
  
```powershell
Out-File -LiteralPath .\abc.txt -InputObject 'The end of the world' -Append

```  
  
[Out-File](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/out-file?view=powershell-5.1)に-Appendオプションを付与して追記を行います。  
  
それ以外はファイルの更新時に[Out-Fileを使用する例](#Out-Fileを使用する例)と同じです。  
  
### System.IO.Fileを利用する方法  
PowerShell5.1以前のSet-Content/Out-FileのEncodingはいくつかのエンコーディングしかサポートしていません。  
そのため、EUCやUTF8のBOMなしを追記する際は.NETのクラスを直接操作する必要があります。  
  
```powershell
	$enc = [System.Text.Encoding]::GetEncoding(20932)
	[System.IO.File]::AppendAllText("c:\dev\ps\file\contenttest\euc_out.txt", "わっふる", $enc)

	$enc = New-Object System.Text.UTF8Encoding($False)
	[System.IO.File]::AppendAllText("c:\dev\ps\file\contenttest\utf8_cout.txt", "わっしょい", $enc)
```  
  
# ファイルの追記の監視  
## 従来の方法  
  
**CentOs7のシェルの例**  
  
```
tail -f log.txt
```  
  
## PowerShellの例  
  
```powershell
Get-Content -Literal log.txt -Wait
```  
  
[Get-Content](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-content?view=powershell-5.1)のWaitオプションで類似のことが行えます。  
  
ただし、UNIXのtailコマンドは複数のファイルを受け付けてファイルの更新を監視しますが、Get-Contentの場合、単一のファイルの監視しかできないようです。  
  
# ファイルを空にする  
## 従来の方法  
  
**Windows10のコマンドプロンプトの例**  
  
```
copy nul test.txt
```  
  
**CentOs7のシェルの例**  
  
```
cp /dev/null test.txt
```  
  
## PowerShellの例  
  
```powershell
Clear-Content -LiteralPath a.txt
Clear-Content -LiteralPath a1.txt,a2.txt
Clear-Content -Path b*.txt
Clear-Content c*.txt -Force

Get-ChildItem * -File | Clear-Content
```  
  
ファイルを削除しないが、内容を空にするには[Clear-Content](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/clear-content?view=powershell-5.1)を使用します。  
  
-LiteralPath,-Pathオプションには表示対象のファイルパスを指定します。  
「,」区切りで複数指定した場合、複数のファイルの内容を表示します。  
  
-Pathオプションを指定した場合、[ワイルドカード](https://github.com/mima3/note/blob/master/PowerShellのメモ～PathとLiteralPathパラメータについて.md#%E3%83%91%E3%83%A9%E3%83%A1%E3%83%BC%E3%82%BF%E3%81%AE%E3%83%AF%E3%82%A4%E3%83%AB%E3%83%89%E3%82%AB%E3%83%BC%E3%83%89)による指定が可能になります。  
  
操作対象のパスはパイプラインの入力から指定も可能です。  
  
読み取り専用ファイルを対象とした場合、規定ではエラーとなります。この場合は-Forceオプションを付与することで上書きが可能になります。  
  
-WhatIfオプションを使用することで事前に更新されるファイルの確認も可能です。  
  
-Confirmオプションを使用することで書き込み前に確認メッセージが表示されます。  
  
-Filter,-Exclude,-Include オプションで対象のフィルタリングが行えます。  
  
# ファイルの属性変更  
## 従来の方法  
  
**Windows10のコマンドプロンプトの例**  
  
```
attrib y.txt +R	
```  
  
**CentOs7のシェルの例**  
  
```
chmod 444 y.txt
```  
  
## PowerShellの例  
  
```powershell
# 読み取り専用にする
Set-ItemProperty  -LiteralPath ./y.txt -Name IsReadOnly -Value $True

# 読み取り専用を解除
Set-ItemProperty -LiteralPath ./y.txt -Name IsReadOnly -Value $False

# 読み取り専用＋システムファイル＋隠しファイルとする
Set-ItemProperty -LiteralPath ./y.txt -Name Attributes -Value 'Hidden,System,ReadOnly'

# アーカイブ属性を付与
Set-ItemProperty -LiteralPath ./y.txt -Name Attributes -Value 'Archive'

# 全ての属性を外す
Set-ItemProperty -LiteralPath ./y.txt -Name Attributes -Value 'Normal'

# ファイルの作成日,更新日,最終アクセス日を変更する
Set-ItemProperty -LiteralPath ./y.txt -Name CreationTime -Value  (Get-Date -Format "yyyy/MM/dd HH:mm
:ss") -Force
Set-ItemProperty -LiteralPath ./y.txt -Name LastWriteTime -Value  (Get-Date -Format "yyyy/MM/dd HH:mm
:ss") -Force
Set-ItemProperty -LiteralPath ./y.txt -Name LastAccessTime -Value  (Get-Date -Format "yyyy/MM/dd HH:mm
:ss") -Force

# 複数ファイルを対象に操作
Set-ItemProperty  -LiteralPath x.txt,y.txt -Name IsReadOnly -Value $True
Set-ItemProperty  -Path *.txt -Name IsReadOnly -Value $True -WhatIf
Set-ItemProperty  -Path *.txt -Name IsReadOnly -Value $True -Confirm

```  
  
[Set-ItemProperty](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/set-itemproperty?view=powershell-5.1)を使用してファイルの属性と更新日、作成日、最終更新日を変更可能です。  
  
-LiteralPath,-Pathオプションには表示対象のファイルパスを指定します。  
「,」区切りで複数指定した場合、複数のファイルの内容を表示します。  
  
-Pathオプションを指定した場合、[ワイルドカード](https://github.com/mima3/note/blob/master/PowerShellのメモ～PathとLiteralPathパラメータについて.md#%E3%83%91%E3%83%A9%E3%83%A1%E3%83%BC%E3%82%BF%E3%81%AE%E3%83%AF%E3%82%A4%E3%83%AB%E3%83%89%E3%82%AB%E3%83%BC%E3%83%89)による指定が可能になります。  
  
操作対象のパスはパイプラインの入力から指定も可能です。  
  
-Nameオプションに与える文字は[FileSystemInfo Class](https://docs.microsoft.com/en-us/dotnet/api/system.io.filesysteminfo?view=netframework-4.8)のプロパティとなります。  
今回使用するものは以下になります。  
  
 - **Attributes**  
    - Archive、Hidden、 Normal、ReadOnly、または System を設定できます。複数設定数場合はカンマ区切りとします。  
 - **CreationTime**  
 - **LastWriteTime**  
 - **LastAccessTime**  
  
-WhatIfオプションを使用することで事前に更新されるファイルの確認も可能です。  
  
-Confirmオプションを使用することで書き込み前に確認メッセージが表示されます。  
  
-Filter,-Exclude,-Include オプションで対象のフィルタリングが行えます。  
  
# ファイルの所有者変更  
## 従来の方法  
  
**Windows10のコマンドプロンプトの例**  
  
```
icacls b.txt /setowner NOTE-MAIN\mima
```  
  
**CentOs7のシェルの例**  
  
```
chown test x.txt
```  
  
## PowerShellの例  
[Hey, Scripting Guy! Windows PowerShell を使用してファイルの所有者を特定する方法はありますか](https://gallery.technet.microsoft.com/scriptcenter/f5a49a45-e667-40e0-9640-555508d8f755)ではファイルの所有者を変更するスクリプトを紹介しています。  
  
```powershell
# 操作対象のファイルのACLを取得して所有者を変更する
$acl = Get-Acl aaa.txt
$user = New-Object System.Security.Principal.NTAccount("NOTE-MAIN", "mima")
$acl.SetOwner($user)
Set-Acl -AclObject $acl -Path aaa.txt
```  
  
まず所有書を変更したいファイルのアクセス制御リスト（ACL)を[Get-Acl](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.security/get-acl?view=powershell-5.1)で取得します。  
  
Get-Aclで取得できるオブジェクトは[System.Security.AccessControl.FileSecurity](https://docs.microsoft.com/en-us/dotnet/api/system.security.accesscontrol.filesecurity?view=netframework-4.8)になり、[SetOwner](https://docs.microsoft.com/en-us/dotnet/api/system.security.accesscontrol.objectsecurity.setowner?view=netframework-4.8#System_Security_AccessControl_ObjectSecurity_SetOwner_System_Security_Principal_IdentityReference_)メソッドで所有者を変更します。  
  
SetOwnerメソッドには[NTAccount](https://docs.microsoft.com/en-us/dotnet/api/system.security.principal.ntaccount?view=netframework-4.8)を与える必要があるのでドメイン名とアカウント名を引数にオブジェクトを作成します。  
  
その後、[Set-Acl](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.security/set-acl?view=powershell-5.1)を使用して所有者を変更したACLオブジェクトを指定のパスに設定します。  
  
Set-AclのLiteralPath、Pathオプションはコンマ区切りで複数のファイルを指定することが可能です。  
また、-Pathオプションを指定した場合、[ワイルドカード](https://github.com/mima3/note/blob/master/PowerShellのメモ～PathとLiteralPathパラメータについて.md#%E3%83%91%E3%83%A9%E3%83%A1%E3%83%BC%E3%82%BF%E3%81%AE%E3%83%AF%E3%82%A4%E3%83%AB%E3%83%89%E3%82%AB%E3%83%BC%E3%83%89)による指定が可能になります。  
  
  
  
  
# ファイル中の文字検索  
## 従来の方法  
  
**Windows10のコマンドプロンプトの例**  
  
```
findstr  /S a ./*
```  
  
**CentOs7のシェルの例**  
  
```
grep -r a ./*
```  
  
## PowerShellの例  
  
```powershell
# カレントディレクトリのtxtファイル中にGetが含まれるものを列挙する
Select-String -Path .\*.txt -Pattern 'Get'

# カレントディレクトリ以下すべてののtxtファイル中にGetが含まれるものを列挙する
Get-ChildItem -LiteralPath . -Filter *.txt -Recurse | Select-String -Pattern 'Get'

# unicode/utf8(BOMなし)/utf(BOMあり)のファイルが検索可能
Select-String -Path .\*.txt -Pattern 'あ'
Select-String -Path .\*.txt -Pattern 'あ' -Encoding UTF8

# unicode/utf(BOMあり)のファイルが検索可能
Select-String -Path .\*.txt -Pattern 'あ' -Encoding Unicode

# cp932/unicode/utf(BOMあり)のファイルが検索可能
Select-String -Path .\*.txt -Pattern 'あ' -Encoding Default

# カレントディレクトリのtxtファイルでファイル名に1を含まないものを対象にGetが含まれるものを列挙する
Select-String -Path .\*.txt -Pattern 'Get' -Exclude *1*

# -NotMatchオプションでPatternに一致しない行を列挙する
Select-String -Path *.txt -Pattern "a" -NotMatch

```  
  
[Select-String](https://docs.microsoft.com/en-us/powershell/module/Microsoft.PowerShell.Utility/Select-String?view=powershell-5.1)はファイル中の文字列を検索します。  
  
  
-LiteralPath,-Pathオプションには表示対象のファイルパスを指定します。  
「,」区切りで複数指定した場合、複数のファイルの内容を表示します。  
  
-Pathオプションを指定した場合、[ワイルドカード](https://github.com/mima3/note/blob/master/PowerShellのメモ～PathとLiteralPathパラメータについて.md#%E3%83%91%E3%83%A9%E3%83%A1%E3%83%BC%E3%82%BF%E3%81%AE%E3%83%AF%E3%82%A4%E3%83%AB%E3%83%89%E3%82%AB%E3%83%BC%E3%83%89)による指定が可能になります。  
  
操作対象のパスはパイプラインの入力から指定も可能です。  
  
-Encodingを指定して文字コードを選択することが可能です。選択したエンコードと実際検索できる内容は[Get-Content時の実験と同じ](#PowerShell5.1+Win10(日本語版） でのEncodingの試験)になります。  
PowerShell6.0より前は-Encodingはコードページの指定ができず、EUC-JPなどは表示できません。また、UTF8もBOM付きしかサポートしていません。  
省略した場合の挙動はPowerShell5.1では「Utf8」となります。これはオンラインヘルプではDefaultと書いてあるので注意してください。  
**※実際の挙動とGet-Helpでは「UTF8」です。**  
  
-Include,-Excludeオプションでファイルのフィルタリングが行えます  
  
-NotMatchオプションを指定した場合、パターンに一致しない行を列挙します。  
  
### 出力結果  
Select-Stringの出力は規定では[Microsoft.PowerShell.Commands.MatchInfo](https://docs.microsoft.com/en-us/dotnet/api/microsoft.powershell.commands.matchinfo?view=powershellsdk-1.1.0)のオブジェクトのセットになります。  
  
```powershell
# 一致した文字のあるファイル名、行番号、行の内容、一致した文字を表示する
Select-String -Path *.txt -Pattern "fa","あ" | % { $_.Filename + " " + $_.LineNumber + " " + $_.Line + " " + $_.Matches }
# euc.txt 2 asdfafaf fa
# sjis.txt 2 asdfafaf fa
# unicode.txt 2 asdfafaf fa
# unicode.txt 3 asfああああ あ

# -Listオプションを付与した場合は、各ファイル1つ見つかったら次行の検査を辞める
Select-String -Path *.txt -Pattern "fa","あ" -List | % { $_.Filename + " " + $_.LineNumber + " " + $_.Line + " " + $_.Matches }
# euc.txt 2 asdfafaf fa
# sjis.txt 2 asdfafaf fa
# unicode.txt 2 asdfafaf fa
```  
  
Quietパラメーターを使用する場合はSystem.Booleanとなり,見つかったか否かだけを返します。  
  
```powershell
Select-String -Path *.txt -Pattern "a" -Quiet
# True
>Select-String -Path *.txt -Pattern "ない文字" -Quiet
# False
```  
  
### AllMatchオプション  
出力結果に一致したパターンを全て含めるかどうかを制御するオプションです。  
  
```text
諸君　私は戦争が好きだ　諸君　私は戦争が好きだ
諸君　私は戦争が大好きだ　殲滅戦が好きだ
電撃戦が好きだ　打撃戦が好きだ　防衛戦が好きだ　包囲戦が好きだ
突破戦が好きだ　退却戦が好きだ　掃討戦が好きだ　撤退戦が好きだ
```  
  
```powershell
# AllMatchオプションを含まない例：
Select-String -LiteralPath sub1/test2.txt -Pattern "諸君","好き" -Encoding Default | % { $_.Filename + " " + $_.LineNumber + " " + $_.Line + " [" + $_.Matches + "]" }
# 出力結果：行あたり一致するパターンを見つけたらその行については検査をやめている。
# test2.txt 1 諸君　私は戦争が好きだ　諸君　私は戦争が好きだ [諸君]
# test2.txt 2 諸君　私は戦争が大好きだ　殲滅戦が好きだ [諸君]
# test2.txt 3 電撃戦が好きだ　打撃戦が好きだ　防衛戦が好きだ　包囲戦が好きだ [好き]
# test2.txt 4 突破戦が好きだ　退却戦が好きだ　掃討戦が好きだ　撤退戦が好きだ [好き]

Select-String -LiteralPath sub1/test2.txt -Pattern "諸君","好き" -Encoding Default -AllMatch | % { $_.Filename + " " + $_.LineNumber + " " + $_.Line + " [" + $_.Matches + "]" }
# 出力結果：行あたり一致するパターンをすべて見つけている。
# test2.txt 1 諸君　私は戦争が好きだ　諸君　私は戦争が好きだ [諸君 諸君]
# test2.txt 2 諸君　私は戦争が大好きだ　殲滅戦が好きだ [諸君]
# test2.txt 3 電撃戦が好きだ　打撃戦が好きだ　防衛戦が好きだ　包囲戦が好きだ [好き 好き 好き 好き]
# test2.txt 4 突破戦が好きだ　退却戦が好きだ　掃討戦が好きだ　撤退戦が好きだ [好き 好き 好き 好き]

```  
  
### Contextオプション  
一致したパターンの前後を表示します。  
以下の形式で指定して、1番目の数値は一致前の行数、2番目の数値は一致後の行数を表示します。  
>-Context 2,3  
  
```powershell
Get-Command | Out-File -FilePath .\Command.txt
Select-String -Path .\Command.txt -Pattern 'Get-Computer' -Context 2, 3

  Command.txt:2680:Cmdlet          Get-CmsMessage                                     3.0.0.0    Microsoft.PowerShell.Security

  Command.txt:2681:Cmdlet          Get-Command                                        3.0.0.0    Microsoft.PowerShell.Core

> Command.txt:2682:Cmdlet          Get-ComputerInfo                                   3.1.0.0    Microsoft.PowerShell.Management

> Command.txt:2683:Cmdlet          Get-ComputerRestorePoint                           3.1.0.0    Microsoft.PowerShell.Management

  Command.txt:2684:Cmdlet          Get-Content                                        3.1.0.0    Microsoft.PowerShell.Management

  Command.txt:2685:Cmdlet          Get-ControlPanelItem                               3.1.0.0    Microsoft.PowerShell.Management

  Command.txt:2686:Cmdlet          Get-Counter                                        3.0.0.0    Microsoft.PowerShell.Diagnostics

```  
  
### Patternオプション  
各行で検索するテキストを設定します。  
-SimpleMatchオプションを同時に使用した場合は単純な文字列で検索します。  
-SimpleMatchオプションを指定しない場合、正規表現となります。  
  
正規表現については以下を参照してください。  
  
 - [about_Regular_Expressions](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_regular_expressions?view=powershell-5.1)  
 - [Regular Expression Language - Quick Reference](https://docs.microsoft.com/en-us/dotnet/standard/base-types/regular-expression-language-quick-reference)  
  
また、規定では大文字と小文字は区別されません。  
区別を行うには、-CaseSensitiveオプションを付与してください。  
  
**実験対象のファイル**  
  
```
aaabb*ccddeeff
bcd
AAABB*CCDDEEFF
BCD
```  
  
```powershell
# 正規表現で検索する例
Select-String -LiteralPath test.txt -Pattern "B*C"
# test.txt:1:aaabb*ccddeeff
# test.txt:2:bcd
# test.txt:3:AAABB*CCDDEEFF
# test.txt:4:BCD

# 文字列で検索する例
Select-String -LiteralPath test.txt -Pattern "B*C" -SimpleMatch
# test.txt:1:aaabb*ccddeeff
# test.txt:3:AAABB*CCDDEEFF

# 大文字小文字区別をする例
Select-String -LiteralPath test.txt -Pattern "B*C" -CaseSensitive -SimpleMatch
# test.txt:3:AAABB*CCDDEEFF
```  
  
# フォルダのサイズ取得  
  
## PowerShellの例  
指定のフォルダ以下のファイルのサイズの合計は以下のようにして取得します。  
  
```powershell
(Get-ChildItem -LiteralPath . -Recurse -Force | Measure-Object -Sum Length).Sum
```  
  
[ファイルの一覧表示](#ファイルの一覧表示)で使用した[Get-ChildItem](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-childitem?view=powershell-5.1)で再帰的にファイルを列挙して[Measure-Object](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/measure-object?view=powershell-6)に渡します。  
  
なお、ジャンクションやシンボリックリンク等を考慮していないので実際のサイズと異なります。  
  
# ファイルの存在チェック  
  
## PowerShellの例  
```powershell
ni test.txt -Force
Test-Path -LiteralPath test.txt
# True

Test-Path -LiteralPath 存在しない.txt
# False

Test-Path -LiteralPath test.txt,存在しない.txt
# True
# False

# ワイルドカードを使用して特定のフォルダでtxt以外があるか調べる
Test-Path -Path c:\dev\ps\file\grep\* -Exclude *.txt

# 特定の日より新しい
 Test-Path -Path t*.txt -NewerThan "2019/09/07 00:00:00" -OlderThan "2019/09/07 10:00:00"

```  
  
指定したパスが存在するかをTrueまたはFalseで返します。  
パスを複数指定した場合、それぞれに対してTrue,Falseを返します。  
  
-Literal,-Pathオプションには対象のファイルパスを指定します。  
「,」区切りで複数指定した場合、複数のファイルの内容を検査します。  
  
-Pathオプションを指定した場合、[ワイルドカード](https://github.com/mima3/note/blob/master/PowerShellのメモ～PathとLiteralPathパラメータについて.md#%E3%83%91%E3%83%A9%E3%83%A1%E3%83%BC%E3%82%BF%E3%81%AE%E3%83%AF%E3%82%A4%E3%83%AB%E3%83%89%E3%82%AB%E3%83%BC%E3%83%89)による指定が可能になります。  
  
-Filter,-Include,-Excludeを使用してさらに絞り込むこともできます。  
  
### よくわからない話  
#### -IsValid  
IsValidオプションは実際にファイルパスの有無をチェックせず構文が正しいかチェックするオプションですが、この動きがよくわからん状態になっています。  
  
**test-path -isValidの判定が全然isValidじゃない件**  
https://plaza.rakuten.co.jp/satocchia/diary/201807100000/  
  
**PowerShell の Test-Path -IsValid が謎くて使えない**  
https://tech.guitarrapc.com/entry/2013/09/18/081112  
  
#### -NewerThanと-OlderThanオプション  
-NewerThanと-OlderThanオプションで日付を指定して存在チェックができますが、ワイルドカードを利用した際の挙動が怪しいです。  
  
以下のようなファイルがあるとします。  
  
```
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----       2019/09/07     22:56                sub1
-a----       2019/09/07     23:47        1613066 Command.txt
-a----       2019/08/31     12:01             24 euc.txt
-a----       2019/09/07     11:50             14 euc_out.txt
-a----       2019/08/31      9:56             26 sjis.txt
-a----       2019/09/08      0:07             37 test.txt
-a----       2019/09/07     22:26              8 test1.txt
-a----       2019/09/07     22:49             10 test2.log
-a----       2019/09/07     22:49             10 test2.txt
-a----       2019/08/31     12:01             42 unicode.txt
-a----       2019/08/31     12:00             28 utf8.txt
-a----       2019/08/31     12:01             31 utf8BOM.txt
-a----       2019/09/07     11:50             23 utf8_cout.txt
```  
  
このフォルダに対して実験をすると以下のようになります。  
  
```powershell
(Get-Item -LiteralPath test.txt).LastWriteTime
# 2019年9月8日 0:07:02

(Get-Item -LiteralPath test.txt).CreationTime
# 2019年9月8日 0:00:57

Test-Path -Path test.txt -NewerThan "2019/09/08 00:07:00"
# Trueとなる

Test-Path -Path t*.txt -NewerThan "2019/09/08 00:07:00"
# Falseとなる

Test-Path -Path t*.txt -NewerThan "2019/09/08 00:00:58"
# Falseとなる

Test-Path -Path t*.txt -NewerThan "2019/09/08 00:00:57"
# Trueとなる
```  
  
つまりワイルドカードを指定しない場合は更新日をみて、ワイルドカードを指定した場合は作成日をみている動きをしています。  
  
ヘルプを見る限りこの挙動が意図したものかどうかはわかりませんでした。  
  
  
# パスの結合  
## PowerShellの例  
  
```powershell
# 以下のケースはいずれも「c:\dev[1]\work」を作成する
Join-Path c:\dev[1] work
Join-Path -Path c:\dev[1] -ChildPath work
C:\dev\ps\file\grep> Join-Path c:\dev[1]\ \work

# 複数同時に作成も可能
Join-Path c:\dev[1],c:\dev[2] work
# c:\dev[1]\work
# c:\dev[2]\work

# -Resolveで実際あるファイルのみ作成する。この場合、-Pathと-ChildPathはワイルドカードを受け付ける
Join-Path c:\win* system* -Resolve
# C:\Windows\System
# C:\Windows\System32
# C:\Windows\SystemApps
# C:\Windows\SystemResources
# C:\Windows\system.ini
```  
  
[Join-Path](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/join-path?view=powershell-5.1)でパスを連結して新しいパスの文字を作成できます。  
  
これによりパスの区切り文字が前後にあるか気にせずパスの作成ができます。  
  
また-Resolveオプションで結合パスの解決を試みます。この際、-Pathと-ChildPathオプションにワイルドカードを指定すると結果が複数になります。  
  
# 相対パスから絶対パスの変換  
## PowerShellの例  
  
```powershell
# 相対パスから絶対パスの変換
Resolve-Path -LiteralPath ../../update
Resolve-Path -Path ../../update
Resolve-Path ../../update
# C:\dev\ps\update

# 絶対パスから相対パスの変換
Resolve-Path -LiteralPath C:\dev\ps\update -Relative
Resolve-Path -Path C:\dev\ps\update -Relative
Resolve-Path C:\dev\ps\update -Relative
# ..\..\update


```  
  
[Resolve-Path](https://docs.microsoft.com/en-us/powershell/module/Microsoft.PowerShell.Management/Resolve-Path?view=powershell-5.1)を利用して相対パスから絶対パスの変換ができます。  
  
# まとめのまとめ  
まとめを作成中に気づいたことや注意した方がいいことを記載しておきます。  
  
 - オンラインヘルプは古いか間違っている可能性があるので挙動が怪しかったらGet-Helpで確認する  
    - 例：PowerShell5.1のSelect-Stringの-Encodingのデフォルト値について  
 - ワイルドカードを使用する更新処理は想定外のファイルを変更する可能性があるので-WhatIfオプションを使用して事前に確認した方がいい。  
 - ワイルドカードを使用する必要がない場合はなるべくLiteralPathを利用する  
    - ファイルシステムで使用できる文字がワイルドカードの文字に含まれているのでエスケープ処理が厄介になる  
 - シンボリックリンクやジャンクションを含むフォルダの扱いについてはリンク先を変更していいかどうかを考えながら慎重に取り扱う。  
 - Encodingをサポートしているコマンドレットは可能な限り明示する。  
    - PowerShell5.1以前は省略した場合の動きがコマンドレット毎に異なる  
 - PowerShell5.1以前はUTF8はBOMなしであるということに留意する。  
    - 特に更新系は下手にクロスプラットフォームで使うファイルを更新するとBOMがついて使えなくなる。.NETのファイル操作の関数を呼び出して更新すること  
 - Aliasは環境ごとに違う。Windowsの場合lsはGet-ItemChildの別名であるが、CentOSでは違う  
    -  「Get-Alias -Definition Get-ChildItem」というようなコマンドで調査できる。  
  
