# 目的  
VOICEROID2の琴葉茜ちゃんをPowerShellでしゃべらせます。  
  
![000097.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/8675bff0-29cf-c440-6e25-2edb70c6a405.jpeg)  
  
この際、なるべくどこでも動くように、UIAutoMationを使用します。  
  
環境：  
  
|  　       |  |  
|:-----------------|-----------:|  
| OS             |Window     |  
| VOICEROIDE2              |2.0.1.0     |  
|PSVersion                  | 5.1.17134.590|  
|PSEdition                  | Desktop|  
|PSCompatibleVersions       | {1.0, 2.0, 3.0, 4.0...}|  
|BuildVersion               | 10.0.17134.590|  
|CLRVersion                 | 4.0.30319.42000|  
|WSManStackVersion          | 3.0|  
|PSRemotingProtocolVersion  | 2.3|  
|SerializationVersion       | 1.1.0.1|  
  
  
   
  
  
  
# 実装しよう  
古式ゆかしいWindowsアプリの実装ではないので、Spyでは茜ちゃんの画面構成を調べることはできません。  
Inspect.exeを使用して、茜ちゃんを隅々まで調べてから実装しましょう。  
  
Inspect  
https://docs.microsoft.com/en-us/windows/desktop/winauto/inspect-objects  
  
## C#での実装  
PowerShellでいきなり実装するのは辛いので、まずC#で実装してみます。  
  
  
```csharp
//using System.Windows.Automation;
// UIAutomationを使用します。
        private void button3_Click(object sender, EventArgs e)
        {
            AutomationElement mainForm = null;
            String message = "茜ちゃん、かわいい！やったー！";
            foreach (var p in Process.GetProcesses())
            {
                if (p.MainWindowTitle.Contains("VOICEROID2"))
                {
                    mainForm = AutomationElement.FromHandle(p.MainWindowHandle);
                }
            }
            if (mainForm == null)
            {
                Debug.WriteLine("起動してない");
                return;
            }
            {
                var elems = mainForm.FindAll(
                    TreeScope.Element | TreeScope.Descendants,
                    new PropertyCondition(AutomationElement.ClassNameProperty, "TextBox"));

                ValuePattern txtboxName = elems[0].GetCurrentPattern(ValuePattern.Pattern) as ValuePattern;
                txtboxName.SetValue(message);
            }
            {
                var elems = mainForm.FindAll(
                    TreeScope.Element | TreeScope.Descendants,
                    new PropertyCondition(AutomationElement.ClassNameProperty, "Button"));

                InvokePattern btn = elems[0].GetCurrentPattern(InvokePattern.Pattern) as InvokePattern;
                btn.Invoke();
            }

            AutomationElementCollection stsMessage;
            do
            {
                Thread.Sleep(500);
                stsMessage = mainForm.FindAll(
                 TreeScope.Element | TreeScope.Descendants,
                    new PropertyCondition(AutomationElement.NameProperty, "テキストの読み上げは完了しました。"));

            } while (stsMessage.Count == 0);
        }
```  
  
※ボタンの増減で動かなくなるコードです  
  
## PowerShellでの実装  
続いてPowerShellで実装します。  
  
**:voiceroid.ps1**  
```PowerShell:voiceroid.ps1
$message = $Args[0]
$target = Get-Process | Where-Object {$_.MainWindowTitle.StartsWith("VOICEROID2") -eq $True} | Select-Object 
if ($target.Length -eq 0) {
  Write-Error "VOICEROID2を起動してください"
  exit 1
}
Add-Type -AssemblyName UIAutomationClient
Add-Type -AssemblyName UIAutomationTypes
$mainForm = [System.Windows.Automation.AutomationElement]::FromHandle($target.MainWindowHandle)


# テキストの検索
$cond = New-Object -TypeName System.Windows.Automation.PropertyCondition([System.Windows.Automation.AutomationElement]::ClassNameProperty, "TextBox")
$elems = $mainForm.FindAll([System.Windows.Automation.TreeScope]::Element -bor [System.Windows.Automation.TreeScope]::Descendants, $cond)
$elem = $elems[0].GetCurrentPattern([System.Windows.Automation.ValuePattern]::Pattern) -as [System.Windows.Automation.ValuePattern]
$elem.SetValue($message);

# ボタン押下
$cond = New-Object -TypeName System.Windows.Automation.PropertyCondition([System.Windows.Automation.AutomationElement]::ClassNameProperty, "Button")
$elems = $mainForm.FindAll([System.Windows.Automation.TreeScope]::Element -bor [System.Windows.Automation.TreeScope]::Descendants, $cond)
$invElm = $elems[0].GetCurrentPattern([System.Windows.Automation.InvokePattern]::Pattern) -as [System.Windows.Automation.InvokePattern]
$invElm.Invoke()


# 読み上げ中は待機
$cond = New-Object -TypeName System.Windows.Automation.PropertyCondition([System.Windows.Automation.AutomationElement]::NameProperty, "テキストの読み上げは完了しました。")
do
{
  Start-Sleep -m 500 
  $elems = $mainForm.FindAll([System.Windows.Automation.TreeScope]::Element -bor [System.Windows.Automation.TreeScope]::Descendants, $cond)
}
while ($elems.Count -eq 0)
```  
  
PowerShellのスクリプトファイルを実行するには管理者権限を用いて実行ポリシーを変更するか、以下のようにバッチファイル経由で実行ポリシーを指定して起動します。  
  
```
powershell -ExecutionPolicy RemoteSigned ./voiceroid.ps1 茜ちゃん！可愛い！やったぁ！
powershell -ExecutionPolicy RemoteSigned ./voiceroid.ps1 働きたくない・・・
powershell -ExecutionPolicy RemoteSigned ./voiceroid.ps1 せやなー
```  
  
  
  
# 問題  
…実は、UIAutomationでは自動操作できない箇所があります。  
ボイスとチューニングにあるタブの中身のコントロールは操作できません。  
  
**UIAutomation で タブの内側の要素が取れない**  
https://teratail.com/questions/53276  
  
上記のページに記載してあるとおりアプリの作り方によって、タブの中身の情報がとれなくなるようです。  
  
  
なお、偉大なる先駆者様はFriendlyというライブラリを使って解決しているようです。  
  
**TTSController**  
https://github.com/mikoto2000/TTSController  
  
**Friendly**  
https://github.com/Codeer-Software/Friendly  
  
