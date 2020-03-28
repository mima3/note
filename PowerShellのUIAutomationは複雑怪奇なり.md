# どぼぢでうごかないのぉおおおお！！C#では動いたでしょう！！！！！  
PowerShellのUIAutomationを使ってウィンドウの操作していると、たまにコントロールの操作に失敗することがあります。  
C#やVB.NET,さらにはVBAでは動作しているのにも関わらず、この事象は発生します。  
  
## 動かないパターン  
まず、Win32で作成した画面を用意します。ここでは以下のような画面を用意しました。  
  
![UIAutomation1png.png](/image/96ad797e-e448-91d2-2230-61848cb36dce.png)　  
  
  
ID1000のEditControlにテキストを入力するようにC#で実装してみます。  
  
```csharp
var root = AutomationElement.RootElement;
var cond1 = new PropertyCondition(AutomationElement.NameProperty, "NativeTest");
var mainForm = root.FindFirst(TreeScope.Element | TreeScope.Children, cond1);

var cond2 = new PropertyCondition(AutomationElement.AutomationIdProperty, "1000");
var edit = mainForm.FindFirst(
    TreeScope.Element | TreeScope.Descendants, 
    cond2);
ValuePattern editValue = edit.GetCurrentPattern(ValuePattern.Pattern) as ValuePattern;
editValue.SetValue("csharsssssp");
```  
  
C#側は上記のコードで期待通り動作します。  
次にPowerShellでC#と同様の実装をして実行してみます。  
  
```powershell:test1.ps1
Add-Type -AssemblyName UIAutomationClient
Add-Type -AssemblyName UIAutomationTypes
$cond = New-Object -TypeName System.Windows.Automation.PropertyCondition([System.Windows.Automation.AutomationElement]::NameProperty, "NativeTest")
$mainForm = $rootElement.FindFirst([System.Windows.Automation.TreeScope]::Element -bor [System.Windows.Automation.TreeScope]::Children, $cond)
$cond = New-Object -TypeName System.Windows.Automation.PropertyCondition([System.Windows.Automation.AutomationElement]::AutomationIdProperty, "1000")
$edit = $mainForm.FindFirst([System.Windows.Automation.TreeScope]::Element -bor [System.Windows.Automation.TreeScope]::Descendants, $cond)
$txt = $edit.GetCurrentPattern([System.Windows.Automation.ValuePattern]::Pattern)
$txt.SetValue("Test2")
```  
  
上記のPowerShellを実行すると下記のエラーが発生します。  
  
```text
"1" 個の引数を指定して "GetCurrentPattern" を呼び出し中に例外が発生しました: "サポートされていないパターンです。"
発生場所 C:\dev\ps\voiceroid\test2.ps1:30 文字:1
+ $txt = $edit.GetCurrentPattern([System.Windows.Automation.ValuePatter ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (:) [], MethodInvocationException
    + FullyQualifiedErrorId : InvalidOperationException
```  
  
この事象はWin32の画面、たとえば、「名前を付けて保存」で表示されるようなコモンダイアログでも発生します。  
~~…VOICEROID2つかって茜ちゃんとｷｬｯｷｬｳﾌﾌしてたらハマった。~~  
  
#### VBAの実装方法  
なおVBAでは以下のように実装します。  
これでも動くので、PowerShellとUIAutomationの相性の問題と考えられます。  
  
```vb
' UIAutomationClient(UIAutomationCore.dll) 参照
Public Sub test()
    Dim uia As UIAutomationClient.CUIAutomation
    Set uia = New UIAutomationClient.CUIAutomation
    
    Dim root As IUIAutomationElement
    Set root = uia.GetRootElement()
    
    Dim cndMainForm As IUIAutomationCondition
    Set cndMainForm = uia.CreatePropertyCondition(UIA_NamePropertyId, "NativeTest")
    
    Dim mainForm As IUIAutomationElement
    Set mainForm = root.FindFirst(TreeScope_Element Or TreeScope_Children, cndMainForm)
    
    Dim cndEdit As IUIAutomationCondition
    Set cndEdit = uia.CreatePropertyCondition(UIA_AutomationIdPropertyId, "1000")
    
    Dim edit As IUIAutomationElement
    Set edit = mainForm.FindFirst(TreeScope_Element Or TreeScope_Descendants, cndEdit)
    
    Debug.Print edit.CurrentClassName

    Dim editValue As IUIAutomationValuePattern
    Set editValue = edit.GetCurrentPattern(UIA_ValuePatternId)
    Call editValue.SetValue("vba set")
End Sub

```  
  
参考：  
**起動中のMicrosoft EdgeからタイトルとURLを取得するVBAマクロ(UI Automation編)**  
https://www.ka-net.org/blog/?p=6076  
  
## なぜエラーとなるのか  
### FindFirstで取得された内容の差異  
まずPowerShellのFindFirstで取得した変数をダンプしてみましょう。  
  
```powershell
## 略
$edit = $mainForm.FindFirst([System.Windows.Automation.TreeScope]::Element -bor [System.Windows.Automation.TreeScope]::Descendants, $cond)
# 変数のダンプ
$edit.Current
#
$txt = $edit.GetCurrentPattern([System.Windows.Automation.ValuePattern]::Pattern)
$txt.SetValue("Test2")

```  
$edit.Currentの値は以下のようになります。  
  
```text
ControlType          : System.Windows.Automation.ControlType
LocalizedControlType : ウィンドウ
Name                 : csharsssssp
AcceleratorKey       :
AccessKey            :
HasKeyboardFocus     : False
IsKeyboardFocusable  : False
IsEnabled            : True
BoundingRectangle    : 428,436,151,33
HelpText             :
IsControlElement     : True
IsContentElement     : True
LabeledBy            :
AutomationId         : 1000
ItemType             :
IsPassword           : False
ClassName            : Edit
NativeWindowHandle   : 6556200
ProcessId            : 9748
IsOffscreen          : False
Orientation          : None
FrameworkId          : Win32
IsRequiredForForm    : False
ItemStatus           :
```  
  
  
次にInspectで該当のコントロールを確認します。  
![UIAutomation2png.png](/image/7ec5b995-a6ac-55d8-6c03-3365e90202e2.png)  
  
LocalizedControlTypeの値がPowerShellでは「ウィンドウ」、Inspectでは「編集」になっています。  
つまり、PowerShellの中では該当コントロールがただのウィンドウのため、ValuePatternに変換できず、エラーとなったと推測できます。  
  
この差異はC#では「UIAutomationClientsideProviders.dll」が読み込まれますが、PowerShellでは読み込まれないことによって生じる差異です。  
  
### 読み込まれたDLLの確認  
VisualStudioでは、デバッグ中にDLLが読み込まれると出力ウィンドウに表示されます。  
これを利用してC#で読み込まれたDLLを確認してみます。  
  
RootElementを取得する直前の出力ウィンドウが以下の状態だとします。  
![UIAutomation3png.png](/image/941fc0b1-ff06-6af9-cbd3-908b482aae61.png)  
  
  
RootElementを取得した時点で以下のように「UIAutomationClientsideProviders.dll」が読み込まれます。  
![UIAutomation4png.png](/image/96cb2198-9ca7-2fdd-9db1-5db2650f7648.png)  
  
  
次にFindFirstを行う前の状態は以下のようになります。変化はありません。  
![UIAutomation5png.png](/image/122276a9-9294-c853-9b5f-796a4cdf3ee1.png)  
  
  
FindFirstを実行することで「UIAutomationClientsideProviders.resources.dll」が読み込まれます。  
![UIAutomation6png.png](/image/e9b02ab0-fcdd-eb5a-0e8f-fdcf2cb51dd6.png)  
  
  
「UIAutomationClientsideProviders.resources.dll」の役割については、おそらく、LocalizedControlTypeを言語環境に合わせて設定するためのリソース情報を扱うためと思われます。  
  
  
次にPowerShellで読み込まれているDLLを確認します。  
これには下記のコマンドを使用します。  
  
```powershell
[System.AppDomain]::CurrentDomain.GetAssemblies()  | Sort-Object -Property FullName | %{$_.FullName + "`t" + $_.Location}
```  
  
Sort-Object以降は出力を見やすくしているため記載しています。  
このコードをPowerShellのGetCurrentPatternの前に入れた際の結果は以下の通りになります。  
  
| FullName   | Location                                        |  
|:-----------------|:----------------------------------------------|  
|Anonymously Hosted DynamicMethods Assembly, Version=0.0.0.0, Culture=neutral, PublicKeyToken=null | |  
|Microsoft.Management.Infrastructure, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35 | C:\WINDOWS\Microsoft.Net\assembly\GAC_MSIL\Microsoft.Management.Infrastructure\v4.0_1.0.0.0__31bf3856ad364e35\Microsoft.Management.Infrastructure.dll|  
|Microsoft.PowerShell.Commands.Utility, Version=3.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35 | C:\WINDOWS\Microsoft.Net\assembly\GAC_MSIL\Microsoft.PowerShell.Commands.Utility\v4.0_3.0.0.0__31bf3856ad364e35\Microsoft.PowerShell.Commands.Utility.dll|  
|Microsoft.PowerShell.ConsoleHost, Version=3.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35 | C:\WINDOWS\Microsoft.Net\assembly\GAC_MSIL\Microsoft.PowerShell.ConsoleHost\v4.0_3.0.0.0__31bf3856ad364e35\Microsoft.PowerShell.ConsoleHost.dll|  
|Microsoft.PowerShell.ConsoleHost.resources, Version=3.0.0.0, Culture=ja, PublicKeyToken=31bf3856ad364e35 | C:\WINDOWS\Microsoft.Net\assembly\GAC_MSIL\Microsoft.PowerShell.ConsoleHost.resources\v4.0_3.0.0.0_ja_31bf3856ad364e35\Microsoft.PowerShell.ConsoleHost.resources.dll|  
|Microsoft.PowerShell.Security, Version=3.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35 | C:\WINDOWS\Microsoft.Net\assembly\GAC_MSIL\Microsoft.PowerShell.Security\v4.0_3.0.0.0__31bf3856ad364e35\Microsoft.PowerShell.Security.dll|  
|mscorlib, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089 | C:\Windows\Microsoft.NET\Framework64\v4.0.30319\mscorlib.dll|  
|mscorlib.resources, Version=4.0.0.0, Culture=ja, PublicKeyToken=b77a5c561934e089 | C:\WINDOWS\Microsoft.Net\assembly\GAC_MSIL\mscorlib.resources\v4.0_4.0.0.0_ja_b77a5c561934e089\mscorlib.resources.dll|  
|System, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089 | C:\WINDOWS\Microsoft.Net\assembly\GAC_MSIL\System\v4.0_4.0.0.0__b77a5c561934e089\System.dll|  
|System.Configuration, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a | C:\WINDOWS\Microsoft.Net\assembly\GAC_MSIL\System.Configuration\v4.0_4.0.0.0__b03f5f7f11d50a3a\System.Configuration.dll|  
|System.Configuration.Install, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a | C:\WINDOWS\Microsoft.Net\assembly\GAC_MSIL\System.Configuration.Install\v4.0_4.0.0.0__b03f5f7f11d50a3a\System.Configuration.Install.dll|  
|System.Core, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089 | C:\WINDOWS\Microsoft.Net\assembly\GAC_MSIL\System.Core\v4.0_4.0.0.0__b77a5c561934e089\System.Core.dll|  
|System.Data, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089 | C:\WINDOWS\Microsoft.Net\assembly\GAC_64\System.Data\v4.0_4.0.0.0__b77a5c561934e089\System.Data.dll|  
|System.DirectoryServices, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a | C:\WINDOWS\Microsoft.Net\assembly\GAC_MSIL\System.DirectoryServices\v4.0_4.0.0.0__b03f5f7f11d50a3a\System.DirectoryServices.dll|  
|System.Management, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a | C:\WINDOWS\Microsoft.Net\assembly\GAC_MSIL\System.Management\v4.0_4.0.0.0__b03f5f7f11d50a3a\System.Management.dll|  
|System.Management.Automation, Version=3.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35 | C:\WINDOWS\Microsoft.Net\assembly\GAC_MSIL\System.Management.Automation\v4.0_3.0.0.0__31bf3856ad364e35\System.Management.Automation.dll|  
|System.Management.Automation.resources, Version=3.0.0.0, Culture=ja, PublicKeyToken=31bf3856ad364e35 | C:\WINDOWS\Microsoft.Net\assembly\GAC_MSIL\System.Management.Automation.resources\v4.0_3.0.0.0_ja_31bf3856ad364e35\System.Management.Automation.resources.dll|  
|System.Numerics, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089 | C:\WINDOWS\Microsoft.Net\assembly\GAC_MSIL\System.Numerics\v4.0_4.0.0.0__b77a5c561934e089\System.Numerics.dll|  
|System.Transactions, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089 | C:\WINDOWS\Microsoft.Net\assembly\GAC_64\System.Transactions\v4.0_4.0.0.0__b77a5c561934e089\System.Transactions.dll|  
|System.Xml, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089 | C:\WINDOWS\Microsoft.Net\assembly\GAC_MSIL\System.Xml\v4.0_4.0.0.0__b77a5c561934e089\System.Xml.dll|  
|UIAutomationClient, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35 | C:\WINDOWS\Microsoft.Net\assembly\GAC_MSIL\UIAutomationClient\v4.0_4.0.0.0__31bf3856ad364e35\UIAutomationClient.dll|  
|UIAutomationProvider, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35 | C:\WINDOWS\Microsoft.Net\assembly\GAC_MSIL\UIAutomationProvider\v4.0_4.0.0.0__31bf3856ad364e35\UIAutomationProvider.dll|  
|UIAutomationTypes, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35 | C:\WINDOWS\Microsoft.Net\assembly\GAC_MSIL\UIAutomationTypes\v4.0_4.0.0.0__31bf3856ad364e35\UIAutomationTypes.dll|  
|WindowsBase, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35 | C:\WINDOWS\Microsoft.Net\assembly\GAC_MSIL\WindowsBase\v4.0_4.0.0.0__31bf3856ad364e35\WindowsBase.dll|  
  
UIAutomationClientsideProvidersが読み込まれていないことが確認できると思います。  
  
### UIAutomationClientsideProvidersってなんだ？  
UIAutomationは以下のようなアーキテクトになっています。  
https://docs.microsoft.com/ja-jp/windows/desktop/WinAuto/architecture-and-interoperability  
  
![UIAutomation7png.png](/image/117dc7a8-a918-21cf-f952-c9a577c5f183.png)  
  
操作されるアプリケーション側をサーバー、アプリケーションを操作する側がクライアントとなります。  
サーバーとクライアントはともにUIAutomationCore.dllをよみこみ、プロセス間通信をもちいてプロパティ値を検索するなどしています。  
  
Win32 コントロールは、UIAutomationClientsideProviders.dll のクライアント側プロバイダーによって Microsoft UI オートメーション に公開されています。   
本来、UIAutomationClientsideProviders.dllは、UI オートメーション クライアント アプリケーションで使用するために、自動的に登録されるはずですが、今回はされなかったため、Win32のコントロールが操作できなかったことになります。  
  
**UI オートメーションによる標準コントロールのサポート**  
https://docs.microsoft.com/ja-jp/dotnet/framework/ui-automation/ui-automation-support-for-standard-controls  
  
  
## 対策  
まず、単純にAdd-Typeで読み込んでもダメでした。  
次に、この事象は何人かの人がはまっており、CodeProjectには、この対応策が記載されています。  
  
**Using WPF UI Automation with PowerShell**  
https://www.codeproject.com/Tips/110824/Using-WPF-UI-Automation-with-PowerShell  
  
```powershell
$source = @"
using System;
using System.Windows.Automation;
namespace UIAutTools
{
    public class Element
    {
        public static AutomationElement RootElement
        {
            get
            {
                return AutomationElement.RootElement;
            }
        }
    }
}
"@
Add-Type -TypeDefinition $source -ReferencedAssemblies( `
    "UIAutomationClient", "UIAutomationTypes")
$root = [UIAutTools.Element]::RootElement
```  
  
これは簡単にいうとPowerShellの型の取り扱いの問題で発生しているっぽいから、C#で記載してしちゃえばいいじゃないという考えです。  
この考えを取り入れて最初のコードを直した物が以下になります。  
  
```powershell:test3.ps1
Add-Type -AssemblyName UIAutomationClient
Add-Type -AssemblyName UIAutomationTypes

$source = @"
using System;
using System.Windows.Automation;
namespace UIAutTools
{
    public class Element
    {
        public static AutomationElement RootElement
        {
            get
            {
                return AutomationElement.RootElement;
            }
        }
    }
}
"@
Add-Type -TypeDefinition $source -ReferencedAssemblies("UIAutomationClient", "UIAutomationTypes")

$rootElement = [UIAutTools.Element]::RootElement


$cond = New-Object -TypeName System.Windows.Automation.PropertyCondition([System.Windows.Automation.AutomationElement]::NameProperty, "NativeTest")
$mainForm = $rootElement.FindFirst([System.Windows.Automation.TreeScope]::Element -bor [System.Windows.Automation.TreeScope]::Children, $cond)


$cond = New-Object -TypeName System.Windows.Automation.PropertyCondition([System.Windows.Automation.AutomationElement]::AutomationIdProperty, "1000")
$edit = $mainForm.FindFirst([System.Windows.Automation.TreeScope]::Element -bor [System.Windows.Automation.TreeScope]::Descendants, $cond)
$edit.Current
$txt = $edit.GetCurrentPattern([System.Windows.Automation.ValuePattern]::Pattern)
$txt.SetValue("Test3")
```  
  
この実装は、Windows7(英語OS)+PowerShell2.0の環境においては期待通り動作しました。  
  
しかしながら、Windows10(日本語OS)+Powershell5.0の環境においては、変わらずUIAutomationClientsideProvidersは読み込まれません。  
そこで、さらに操作対象のウィンドウをFindFirstで取得するところまでC#で記載します。  
※ちょうどC#でUIAutomationClientsideProvidersResourceまで読み込んだ時点です。  
  
```powershell:test4.ps1
Add-Type -AssemblyName UIAutomationClient
Add-Type -AssemblyName UIAutomationTypes

$source = @"
using System;
using System.Windows.Automation;
namespace UIAutTools
{
    public class Element
    {
        public static AutomationElement RootElement
        {
            get
            {
                return AutomationElement.RootElement;
            }
        }
        public static AutomationElement GetMainWindowByTitle(string title) {
            PropertyCondition cond = new System.Windows.Automation.PropertyCondition(System.Windows.Automation.AutomationElement.NameProperty, title);
            return RootElement.FindFirst(TreeScope.Element | TreeScope.Children, cond);
        }
    }
}
"@
Add-Type -TypeDefinition $source -ReferencedAssemblies("UIAutomationClient", "UIAutomationTypes")

$mainForm = [UIAutTools.Element]::GetMainWindowByTitle("NativeTest")


$cond = New-Object -TypeName System.Windows.Automation.PropertyCondition([System.Windows.Automation.AutomationElement]::AutomationIdProperty, "1000")
$edit = $mainForm.FindFirst([System.Windows.Automation.TreeScope]::Element -bor [System.Windows.Automation.TreeScope]::Descendants, $cond)
$edit.Current
$txt = $edit.GetCurrentPattern([System.Windows.Automation.ValuePattern]::Pattern)
$txt.SetValue("Test4")
```  
  
これにより、UIAutomationClientsideProvidersが読み込まれ、期待通り動作します。  
  
**注意**  
Add-TypeでC#のコードを追加した場合、PowerShellを起動しなおさないかぎり、そのコードは残り続けます。  
  
また、その影響か、一度、うまくいかないコードを動かすと、正しく動くコードを実行してもPowerShellを起動しなおさない限り正常に動作しません。  
  
  
  
# まとめ  
UIAutomationを使用する際、Win32やWPFといった古いウィンドウを操作する場合、C#として実装しないと正常に動作しない場合があります。  
また、新しめのアプリケーションでもメッセージボックスや名前を付けて保存などの共通のダイアログはWin32で表示される場合があるので、今回の事象に遭遇する場合があります。  
  
また、PowerShellのUIAutomationのライブラリは存在してますが、実は基本部分をC#で実装してDLLを提供している形なので、今回の罠にはハマりません。  
https://archive.codeplex.com/?p=uiautomation  
  
なお、何故PowerShellだとUIAutomationClientsideProvidersが呼ばれないかはソースを見てみましたが全くわからんかったです。  
  
AutomationElement.cs  
https://referencesource.microsoft.com/#UIAutomationClient/System/Windows/Automation/AutomationElement.cs  
  
UiaCoreAPI.cs : このAPI経由でUIAutomationCore.dllを操作している。  
https://referencesource.microsoft.com/#UIAutomationClient/MS/Internal/Automation/UiaCoreAPI.cs  
  
PowerShellのコード  
https://github.com/PowerShell/PowerShell  
