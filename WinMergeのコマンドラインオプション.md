# 前書き  
SIerをディスる記事が登場する度、流れ弾が飛んでくるWinMergeですが、実際のところ、その機能を十分つかいこなせている現場は少ないです。  
  
たとえば、WinMergeはコマンドラインから実行できます。これにより外部プログラムの実行が容易になっているのですが、これはあまり使いこなせていません。  
https://manual.winmerge.org/Command_line.html  
  
実はドキュメント化されていないパラメータがいくつかあります。  
たとえば、「-or」というレポートを出力するパラメータについては実装はされていますが、ドキュメント化されてはいません。  
https://github.com/WinMerge/winmerge/blob/30a5674030a8318a9239893a3cc0a4e8798a6256/Src/MergeCmdLineInfo.cpp  
  
  
  
# コマンドラインのパラメータ  
2.16.4.0でサポートしているパラメータは下記のようになります。  
これは下記を参考に作成しています。  
・ヘルプ  
・https://github.com/WinMerge/winmerge/blob/30a5674030a8318a9239893a3cc0a4e8798a6256/Src/MergeCmdLineInfo.cpp  
・https://sourceforge.net/p/winmerge/support-requests/109/  
  
|オプション|説明|  
|:-----|:----|  
|-?|WinMergeヘルプを開きます|  
|-o "outputfilename"|マージした結果のファイルを保存するオプションの出力フォルダを指定します。 もし、出力パスを指定した場合、あるペインを変更後保存すると、変更は出力パスのファイルに保存され、 元ファイルは前の状態のままになります。 |  
|-or "reportfilename"|レポートの出力を出力するパスを指定します。この機能はドキュメントにのっていません。|  
|dl "desc"|左側タイトルバーの説明を指定します。 デフォルトのフォルダやファイル名テキストに上書きされます。例: /dl "Version 1.0" や /dl WorkingCopy. スペースを含む説明はダブルクォーテーションマークで括ってください。 |  
|-dm "desc"| /dlと同様に中央タイトルバーの説明を指定します。|  
|-dr "desc" |/dlと同様に右側タイトルバーの説明を指定します。 |  
|-e|EscキーでWinMergeが閉じるようにします。 WinMergeを外部比較アプリケーションとして使用する場合に便利です。 (ダイアログのようにすばやくWinMergeを閉じることができます) この引数を指定しなかった場合、すべてのウインドウを閉じるのに何回もEscキーを 押さなければならないことになるかもしれません。(2つ以上タブが開かれている場合、一回のESCキーの押下でWinMergeが終了してしまうのを期待している人はいないような気がしたので、日本語版ではこのオプションを指定しても2つ以上タブが開かれている場合は1つのタブを閉じるだけにし、タブが1つの時またはタブが一つもない時にWinMergeを終了するようにしました)|  
|-f "mask"|比較を制限するために、指定したフィルタを適用します。 フィルタは*.h *.cppのようなファイルマスクか、 XML/HTML Develのようなファイルフィルタの名前です。 スペースを含むフィルタマスクやフィルタ名はダブルクォーテーションマークで括ってください|  
|-r|すべてのサブフォルダ内のすべてのファイルを比較します(再帰比較)。 ユニークフォルダ (片方のみ存在するフォルダ)は、分離された項目として比較結果内にリストされます。 サブフォルダまで含めるとかなり比較時間が増大してしまうことに注意してください。 このパラメータを指定しなかった場合、WinMergeは比較するフォルダ内のファイルとトップレベルのサブフォルダのみリストします。 サブフォルダの中までは比較しません。|  
|-s|WinMergeウインドウを1つのインスタンスに制限します。 例えば、WinMergeが既に実行中ならば、新しい比較は同じインスタンス内で実行されます。 この引数を指定しなかった場合、複数のウインドウが開かれる可能性があります: 設定によっては、新しい比較が既に存在するウインドウで実行されることも新しいウインドウで 実行されることもあります。|  
|-noninteractive|比較やレポート出力後にWinMergeを終了します。この機能はドキュメントにのっていません。|  
|-noprefs|レジストリの読み書きをしません。設定値はデフォルトを使用します。この機能はドキュメントにのっていません。|  
|-minimize|minimize 最小化状態でWinMergeを開始します。 このオプションは長時間かかる比較を行う場合に便利です。|  
|-maximize|最大化状態でWinMergeを開始します。|  
|-prediffer　predifferプラグイン|指定されている場合はpredifferプラグインを取得します（それ以外の場合、predifferプラグインは空白になります。これはデフォルトです）。この機能はドキュメントにのっていません。|  
|-wl|読み取り専用として左側を開きます。 比較時、左側を変更したくない場合に使用してください。|  
|-wm|読み取り専用として中央を開きます。 比較時、中央を変更したくない場合に使用してください。|  
|-wr|読み取り専用として右側を開きます。 比較時、右側を変更したくない場合に使用してください|  
|-ul|左側パスが最近使用した項目(MRU)リストに追加されるのを防ぎます。 外部アプリケーションは、ファイルまたはフォルダの選択ダイアログのMRUリストにパスを 追加するべきではありません|  
|-um|中央のパスが最近使用した項目(MRU)リストに追加されるのを防ぎます。 外部アプリケーションは、ファイルまたはフォルダの選択ダイアログのMRUリストにパスを 追加するべきではありません。|  
|-ur|右側パスが最近使用した項目(MRU)リストに追加されるのを防ぎます。 外部アプリケーションは、ファイルまたはフォルダの選択ダイアログのMRUリストにパスを 追加するべきではありません。|  
|-u または -ub(非推奨）|/u (または/ub) 各々(左、右、中央)のパスが最近使用した項目(MRU)リストに追加されるのを防ぎます。 外部アプリケーションは、ファイルまたはフォルダの選択ダイアログのMRUリストにパスを 追加するべきではありません。 |  
|-fl|起動時、左側にフォーカスを当てます|  
|-fm|起動時、中央にフォーカスを当てます|  
|-fr|起動時、右側にフォーカスを当てます|  
|-al|起動時、左側で自動マージします。|  
|-am|起動時、中央で自動マージします。|  
|-ar|起動時、右側で自動マージします。|  
|-x|同一ファイルの比較をしたときにWinMergeを閉じます。 (情報ダイアログを表示した後) このパラメータは比較後に効果がなくなります。 例えば、もしファイルがマージか編集の結果として同一となった場合です。 このパラメータは、WinMergeを外部アプリケーションとして使用したり、 差異のないファイルを無視することによって余分なステップを取り除きたい場合に便利です。 |  
|-xq|-x に似ていますが、同一ファイルであってもメッセージボックスを表示しません|  
|-cp コードページ|比較時のコードページを指定します。この機能はドキュメントにのっていません。|  
|-ignorews|空白を無視します。この機能はドキュメントにのっていません。|  
|-ignoreblanklines|空白行を無視します。この機能はドキュメントにのっていません。|  
|-ignorecase|大文字小文字を無視します。この機能はドキュメントにのっていません。|  
|-ignoreeol|改行の無視します。この機能はドキュメントにのっていません。|  
|-ignorecodepage|コードページの違いを無視します。この機能はドキュメントにのっていません。|  
|-cfg 設定情報または-config 設定情報|設定情報をコマンドラインから指定します。この機能はドキュメントにのっていません。詳細は[設定情報をコマンドラインで渡す])#設定情報をコマンドラインで渡す)を参照してください|  
  
  
## 設定情報をコマンドラインで渡す  
以下のような記載で設定情報をコマンドラインから渡すことが可能です。  
  
```
"C:\Program Files\WinMerge\WinMergeU.exe" C:\dev\winmerge\a.txt C:\dev\winmerge\b.txt -cfg "Font/Height=32" -cfg "Font/FaceName=ＭＳ 明朝" -cfg "Font/Underline=1"
```  
  
![image.png](/image/23915577-61b5-3972-9d09-035dbabb1951.png)  
  
-cfgで指定できる設定項目の種類は設定値からエクスポートしたものと同じになります。  
  
・編集→設定でひらくオプション画面で「エクスポート」を行う  
![image.png](/image/026d5121-1552-03b5-3082-06cd3c913e90.png)  
  
・作成されるINIファイルは下記の通り  
![image.png](/image/c604b5b8-ed50-0f89-3ed4-6b9e10494d48.png)  
  
## ファイルの比較結果をレポートとして出力する例  
/orを使用してレポートにファイルを出力します。  
この際、同時に/noninteractiveを指定してレポート出力後にWinMergeを終了します。  
  
```
"C:\Program Files\WinMerge\WinMergeU.exe" C:\dev\winmerge\test1.txt C:\dev\winmerge\test2.txt  /minimize /noninteractive /u /or C:\dev\winmerge\out.html
```  
  
**作成されたレポート**  
![image.png](/image/ad905297-76f4-1b8f-f269-cf72479749ce.png)  
  
  
  
## フォルダの比較結果をレポートとして出力する例  
-orを使用してレポートにファイルを出力します。  
-noninteractiveを指定してレポート出力後にWinMergeを終了します。  
-rを使用してサブフォルダを含めて比較を行います。  
-cfgを指定して下記の設定を適用します。  
　Settings/DirViewExpandSubdirs=1 …オプションの比較>フォルダー>自動的にサブフォルダーを展開する  
　ReportFiles/ReportType=2 …フォルダ比較レポート＞スタイル：シンプルなHTML形式  
　ReportFiles/IncludeFileCmpReport=1 …フォルダ比較レポート＞ファイル比較レポートを含める  
  
  
```
"C:\Program Files\WinMerge\WinMergeU.exe" C:\dev\winmerge\abc C:\dev\winmerge\acc -minimize -noninteractive -noprefs -cfg Settings/DirViewExpandSubdirs=1 -cfg ReportFiles/ReportType=2 -cfg ReportFiles/IncludeFileCmpReport=1 -r -u -or C:\dev\winmerge\out2.html
```  
  
**作成されたレポート（ディレクトリの比較結果)**  
![image.png](/image/0be867c1-2c4b-da91-9802-bd9042bcb535.png)  
  
※注意：2.14だと-minimizeがついているとレポートが常に差分なしになってしまう。  
  
## コマンドラインからマージする例  
  
```
"C:\Program Files\WinMerge\WinMergeU.exe" base.c remote.c local.c /ar /u /o C:\dev\winmerge\out.c /noninteractive
```  
  
  
残念ながら、以下で止まるのでコマンドラインだけの操作で自動化はできません。  
またコンフリクトが発生した場合も、マージできません。  
  
![image.png](/image/6f005c27-667a-24f0-fcbb-05ef7a9313a9.png)  
  
どうしても自動化したかったら以下のPowerShellを使います。  
  
```powershell:automerge.ps1
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
        var rootCond = new PropertyCondition(AutomationElement.ClassNameProperty, "WinMergeWindowClassW");
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


# 5秒間ダイアログがでるのを待機しているが、あまりよくない。
$dialog = [AutomationHelper]::WaitWindowByTitle("変更されたファイルを保存しますか?", 5)
if ($dialog -eq $null) {
    Write-Host "マージできませんでした"
} else {
    pushButtonById $dialog "1"
    Write-Host "マージでしました"
}


```  
  
このPowerShellを以下のような感じで実行します。  
  
```
start cmd /c "C:\Program Files\WinMerge\WinMergeU.exe" base.c remote.c local.c /ar /u /o C:\dev\winmerge\out.c /noninteractive
powershell -ExecutionPolicy RemoteSigned -File  ./automerge.ps1

```  
  
## タイトルバーの説明　/dl /dm /dr  
/dl /dm /drはタイトルバーに説明を追加します。  
  
```
WinMergeU.exe C:\dev\winmerge\test1.txt C:\dev\winmerge\test2.txt C:\dev\winmerge\test3.txt /dl "Version 1.0" /dm "Version 1.1" /dr "Version 1.2"
```  
  
![image.png](/image/248d68ef-4a6d-e7e8-75bf-69da97e4825f.png)  
  
## ファイルフィルタの設定 /f  
指定したフィルタを設定して比較対象のファイルを制限します。  
スペースで区切ることで複数のファイルを指定することが可能です。  
  
以下の例ではcppとhのみをフィルタした結果となります。  
  
```
WinMergeU.exe C:\dev\winmerge\abc C:\dev\winmerge\acc -f "*.cpp *.h"
```  
  
![image.png](/image/e1e98f83-5fe5-04ff-9c2d-a863e6196f93.png)  
※他の拡張子が表示されない  
  
## 末尾の空白の無視 /ignorews  
末尾の半角スペースとタブを無視します。  
  
空白を無視しない場合：  
![image.png](/image/59b7eb05-a17e-a5d7-5182-6238a40a2381.png)  
  
  
空白を無視する場合:  
  
```
"C:\Program Files\WinMerge\WinMergeU.exe" C:\dev\winmerge\a.txt C:\dev\winmerge\b.txt  /ignorews
```  
  
![image.png](/image/9ae6c907-42e6-a725-767d-ca4502c68be6.png)  
  
## 空白行の無視 /ignoreblanklines  
ignoreblanklinesオプションを使用することで空白行の差異を無視することができます。  
  
  
空白行を無視しない  
![image.png](/image/dbbd434a-1cf4-e363-701a-9d7927670e13.png)  
  
  
空白行を無視する  
  
```
"C:\Program Files\WinMerge\WinMergeU.exe" C:\dev\winmerge\a.txt C:\dev\winmerge\b.txt  /ignoreblanklines
```  
  
![image.png](/image/3e142ee9-5119-faad-7e26-0cae740a10d5.png)  
  
## 大文字小文字を無視　/ignorecase  
ignorecaseオプションを利用することで大文字小文字の差異を無視することが可能です。  
  
無視しない場合  
![image.png](/image/7118c58e-a360-3e3b-3f22-c5dea9097047.png)  
  
無視する場合  
  
```
"C:\Program Files\WinMerge\WinMergeU.exe" C:\dev\winmerge\c.txt C:\dev\winmerge\d.txt   /ignorecase
```  
  
![image.png](/image/20084fc5-48de-689f-6a20-c4fecd49d0c0.png)  
  
  
## 改行文字の無視 /ignoreeol  
行末の改行文字の違いを無視します。  
  
無視しない場合  
![image.png](/image/816fda6f-586e-f97a-411d-b9eee447f13d.png)  
  
  
無視する場合  
  
```
"C:\Program Files\WinMerge\WinMergeU.exe" C:\dev\winmerge\a.txt C:\dev\winmerge\b.txt  /ignoreeol
```  
  
![image.png](/image/7ec1711b-e6c1-188b-d136-bab3f3f04b21.png)  
  
## コードページの無視 /ignorecodepage  
/ignorecodepageでコードページの差異を無視できます。  
ただし、2.16.4.0で実験した限り、このパラメータを無効にしてかつ、下記の設定であっても、コードページの違いを無視して比較しているようです。  
  
![image.png](/image/00c7f9c1-52ad-6ced-6396-dc50b92a0511.png)  
  
![image.png](/image/cd45bc3a-0235-b125-1552-f5a2baab81d5.png)  
※コードページの違いを無視する設定で、文字コードのことなるファイルを比較しても差異がでてこない。  
  
  
# まとめ  
コマンドラインを利用することで自動でレポートの作成が行えます。  
ただし、パッチの作成や、マージ結果の作成は自動でできないようです。  
