# はじめに  
UiPathでPowerShellを起動することを考えてみます。  
  
アクティビティの一覧を見ていると２つほど案が思いつくと思います。  
  
１つ目は[StartProcessアクティビティ](https://docs.uipath.com/activities/docs/start-process)でプロセスを開始する方法  
２つ目は[InvokePowerShellアクティビティ](https://docs.uipath.com/activities/lang-ja/docs/invoke-power-shell)で実行する方法です。  
  
今回は以下のようなPowerShellを実行する方法を考えてみます。  
  
**:list.ps1**  
```powershell:list.ps1
Param(
    [String]$Path
)
$ErrorActionPreference="Stop"
if ($Path -eq "") {
    Set-Content C:\share\test\result.txt -Value "error "
    throw "Error"
}
try {
    $ret = Get-ChildItem -Path $Path -Filter *.txt
    Set-Content C:\share\test\result.txt -Value $ret
    return $ret
} catch {
    Set-Content C:\share\test\result.txt -Value $_
    throw "Error"
}
```  
  
# 動作環境  
・UiPath Community Edition 2019.6.0  
・C#プロジェクト  
・Windows 10  
・PowerShell5.1  
  
# StartProcessアクティビティ  
単純な例では以下のようなフローになります。  
![image.png](/image/55caf3e1-fa7b-8208-23a4-01702be8b4ee.png)  
  
**プロセスを開始**  
  
|プロパティ名|設定値|  
|:-------|:-----|  
|ファイル名|@"C:\Windows\System32\WindowsPowerShell\v1.0\powershell"|  
|引数|@"-ExecutionPolicy RemoteSigned -File C:\share\test\list.ps1 C:\share\"|  
  
これでPowerShellが起動してスクリプトが実行されます。  
しかし、[StartProcessアクティビティ](https://docs.uipath.com/activities/docs/start-process)は非同期な処理でプロセスの終了を待ちません。  
このことは、PowerShellスクリプト実行後に後続の処理がある場合に問題になります。  
※list.ps1の例だと作成されたresult.txtを利用してなにかをする場合  
  
この問題の回避策は下記で議論されています。  
  
**Make Start Process wait until the end of the command in cmd**  
https://forum.uipath.com/t/make-start-process-wait-until-the-end-of-the-command-in-cmd/15551  
  
ここで挙げられた回避策は以下の３つになります。  
  
 - 案１：画面要素を監視してスクリプトの終了を検知する  
 - 案２：もしファイルを出力するスクリプトであるなら、そのファイルの更新を検知する  
 - 案３：PowerShellのWait-Process を使用してプロセスの終了を検知する。  
  
案１は[OnUiElementVanish](https://docs.uipath.com/activities/lang-ja/docs/on-ui-element-vanish)アクティビティを使用することになりますが、非同期で動作しているためこのアクティビティに到達した時点で、すでに画面が終了している可能性もありますし、逆にこのアクティビティを通過してしまった後に画面が表示される可能性があります。  
  
案２は[MonitorEvents](https://docs.uipath.com/activities/docs/monitor-events)アクティビティ内に[FileChangeTrigger](https://docs.uipath.com/activities/docs/file-change-trigger)アクティビティを配置してファイルの最終更新日やサイズを監視して変化があったら終了とみなす方法ですが、これもイベント監視の前にファイルの操作が終わる可能性があります。  
  
案３は[StartProcessアクティビティ](https://docs.uipath.com/activities/docs/start-process)で対応する方法でなく、[InvokePowerShellアクティビティ](https://docs.uipath.com/activities/lang-ja/docs/invoke-power-shell)で対応する方法になります。  
  
つまり結論を述べると、[StartProcessアクティビティ](https://docs.uipath.com/activities/docs/start-process)でPowerShellを起動して終了を待つのは適していません。  
  
# InvokePowerShellアクティビティ  
  
## TypeArgumentプロパティについての注意  
[InvokePowerShellアクティビティ](https://docs.uipath.com/activities/lang-ja/docs/invoke-power-shell)を使用する場合、TypeArgumentプロパティには注意をする必要があります。  
  
これはスクリプト内で返却されたオブジェクトの型にキャストできるものでなければなりません。  
たとえば、「コマンドテキスト」プロパティに以下のコマンドレットを記述したとします。  
  
```powershell
Get-ChildItem
```  
  
[FileInfo](https://docs.microsoft.com/en-us/dotnet/api/system.io.fileinfo?view=netframework-4.8)と[DirectoryInfo](https://docs.microsoft.com/en-us/dotnet/api/system.io.directoryinfo?view=netframework-4.8)のコレクションが返却されます。TypeArgumentプロパティにはコレクション内の型がキャスト可能な値、すなわちベースクラスであるObjectや[FileSystemInfo](https://docs.microsoft.com/en-us/dotnet/api/system.io.filesysteminfo?view=netframework-4.8)を指定する必要があります。  
  
もしこのことを守らないと以下のようなエラーが発生します。  
![image.png](/image/c9396691-30d5-04b5-a368-ba1b1cbc66a5.png)  
  
  
次に、以下のようなコマンドレットを記述します。  
  
```powershell
Get-Process
Get-ChildItem
```  
  
またスクリプトの入力にはチェックを付けてください。  
  
![image.png](/image/e9f784c7-a1bc-083b-bc07-ce161909e30f.png)  
  
これを実行すると、System.Diagnostics.Processのアイテムが続いたあとに、[FileInfo](https://docs.microsoft.com/en-us/dotnet/api/system.io.fileinfo?view=netframework-4.8)と[DirectoryInfo](https://docs.microsoft.com/en-us/dotnet/api/system.io.directoryinfo?view=netframework-4.8)のアイテムが続くコレクションが返されます。  
  
つまり、スクリプト内で出力されたオブジェクトは全て受け取れることを意味しており、また、そのオブジェクトを受け入れられる型をTypeArgumentに指定する必要があります。  
  
## PowerShellのスクリプトファイルパスを指定  
以下にlist.ps1のPowerShellのスクリプトを実行してその結果をデバッグ出力するサンプルを記載します。  
  
![image.png](/image/2e71fbe9-54db-215b-585c-94936009fe23.png)  
  
**PowerShellを呼び出し**  
  
|プロパティ名|設定値|  
|:-------|:-----|  
|TypeArgument|System.IO.FileSystemInfo|  
|スクリプト入力|チェックON|  
|コマンドテキスト|@"C:\share\test\list.ps1 C:\share\"|  
|パラメータ|なし|  
|出力|result※ctrl+kで変数を作成|  
  
**繰り返し（コレクションの各要素)**  
  
|プロパティ名|設定値|  
|:-------|:-----|  
|TypeArgument|System.IO.FileSystemInfo|  
|コレクション値|result|  
  
![image.png](/image/bef3127c-1f63-ef44-face-c912aacda04d.png)  
  
**1行を書き込み**  
  
|プロパティ名|設定値|  
|:-------|:-----|  
|テキスト|item.FullName|  
  
  
[InvokePowerShellアクティビティ](https://docs.uipath.com/activities/lang-ja/docs/invoke-power-shell)の「コマンドテキスト」プロパティにはファイル名を与えることができます。  
この場合、「パラメータ」プロパティによるスクリプトによるパラメータの設定はできないようなので、「コマンドテキスト」プロパティのスクリプト名に続けてスクリプトに与えたいパラメータを記載します。  
  
## PowerShellのスクリプトの内容を指定  
スクリプトの内容をあらかじめ読み込んでおいて、実行することも可能です。  
この場合、「パラメータ」プロパティを利用してPowerShellのスクリプトのパラメータを指定することができます。  
PowerShellのスクリプトファイルパスを指定する場合と異なり、.NETの型をそのまま渡せるメリットがあります。  
  
![image.png](/image/2d945e26-fd4f-2bf0-8b26-34352d15bd2a.png)  
  
**テキストファイルを読み込む**  
  
|プロパティ名|設定値|  
|:-------|:-----|  
|ファイル名|@"C:\share\test\list.ps1"|  
|出力|script※ctrl+kで作成|  
  
  
**PowerShellを呼び出し**  
  
|プロパティ名|設定値|  
|:-------|:-----|  
|TypeArgument|System.IO.FileSystemInfo|  
|スクリプト入力|チェックON|  
|コマンドテキスト|script|  
|パラメータ|名前：Path 型：String 値：@"C:\share"|  
|出力|result※ctrl+kで変数を作成|  
  
**繰り返し（コレクションの各要素)**  
PowerShellのスクリプトファイルパスを指定する場合と同じ  
  
# まとめ  
UiPathでPowerShellを起動する場合は[InvokePowerShellアクティビティ](https://docs.uipath.com/activities/lang-ja/docs/invoke-power-shell)を使用します。  
ファイルパスを直接与えるか、一旦、スクリプトファイルを読み込んでから渡すかは、パラメータを文字で渡すかオブジェクトで渡すかで検討した方がいいでしょう。  
