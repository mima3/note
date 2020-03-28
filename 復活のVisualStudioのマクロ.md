# はじめに  
VisualStudio2008あたりまではVisualStudioにマクロがついていました。  
この機能は最近のVisualStudioからは削除されましたが、再び拡張機能として復活しています。  
  
https://github.com/microsoft/VS-Macros  
  
VisualStudio2019でもそれっぽく動作するようです。  
  
# 導入手順  
(1)拡張機能の管理より「Macros for Visual Studio」をインストールします  
![image.png](/image/6161ade5-cac4-9203-22cc-5d56c3634de3.png)  
  
(2)インストール後、再起動をするとツールからMacrosメニューが使用できるようになります。  
![image.png](/image/bae0147c-1767-1f77-5ed7-655edd397178.png)  
  
  
  
(3)Macro Exploreを選択すると、Macro Exploreウィンドウが表示されます。  
![image.png](/image/f44d727f-800d-0086-0a31-db3f8dc2bf26.png)  
  
(4)JSファイルを右クリックして開くを選択することでJSの中身が編集可能になります。  
![image.png](/image/343a7f67-1e73-ceb5-5f23-2348e22f50e2.png)  
  
なお、JSファイルをダブルクリックすることで記載したJSを実行可能です。  
  
# サンプル  
## 出力ウィンドウに文字列を出力する方法  
  
```js
/// <reference path="C:\Users\ユーザー名\AppData\Local\Microsoft\VisualStudio\16.0_ea48cace\Macros\dte.js" />
// https://github.com/Microsoft/VS-Macros/issues/28
var outputWindowPane = dte.Windows.Item("{34E76E81-EE4A-11D0-AE2E-00A0C90FFFC3}").Object.ActivePane;
dte.Windows.Item("{34E76E81-EE4A-11D0-AE2E-00A0C90FFFC3}").Activate();
outputWindowPane.OutputString("display this text in the output window panel\n");
```  
  
![image.png](/image/89f04403-af5f-c826-0ead-361547f46d99.png)  
  
## 出力ウィンドウのクリアを行う  
  
```js
dte.ExecuteCommand("Edit.ClearAll");
```  
  
## ソリューションのプロジェクトのアイテムをすべて列挙する方法  
  
```js
/// <reference path="C:\Users\ユーザー名\AppData\Local\Microsoft\VisualStudio\16.0_ea48cace\Macros\dte.js" />
var outputWindowPane = dte.Windows.Item("{34E76E81-EE4A-11D0-AE2E-00A0C90FFFC3}").Object.ActivePane;
dte.Windows.Item("{34E76E81-EE4A-11D0-AE2E-00A0C90FFFC3}").Activate();


function dumpProjectItems(projectItems, lv) {
    for (var i = 1; i <= projectItems.Count; i++) {
        var file = projectItems.Item(i);
        for (var j = 0; j < lv; ++j) {
            outputWindowPane.OutputString(' ');
        }
        outputWindowPane.OutputString(file + "\n");

        if (file.SubProject != null) {
            dumpProjectItems(file.ProjectItems, lv + 1);
        } else if (file.ProjectItems != null && file.ProjectItems.Count > 0) {
            dumpProjectItems(file.ProjectItems, lv + 1);
        }
    }
}

for (var i = 1; i <= dte.Solution.Projects.Count; i++) {
    outputWindowPane.OutputString(dte.Solution.Projects.Item(i).Name + "\n");
    dumpProjectItems(dte.Solution.Projects.Item(i).ProjectItems, 1);
}
```  
## 関数の先頭に移動してその関数の情報を取得  
  
```js
/// <reference path="C:\Users\ユーザー名\AppData\Local\Microsoft\VisualStudio\16.0_ea48cace\Macros\dte.js" />
var outputWindowPane = dte.Windows.Item("{34E76E81-EE4A-11D0-AE2E-00A0C90FFFC3}").Object.ActivePane;
dte.Windows.Item("{34E76E81-EE4A-11D0-AE2E-00A0C90FFFC3}").Activate();

var textSelection = dte.ActiveDocument.Selection;

// Define Visual Studio constants
var vsCMElementFunction = 2;
var vsCMPartHeader = 4;

var codeElement = textSelection.ActivePoint.CodeElement(vsCMElementFunction);

if (codeElement != null) {
    // 関数の先頭に移動
    textSelection.MoveToPoint(codeElement.GetStartPoint(vsCMPartHeader));
    dte.ActiveDocument.Activate();

    outputWindowPane.OutputString(codeElement.FullName + "\n");
    outputWindowPane.OutputString(codeElement.GetStartPoint(vsCMPartHeader).Line + "\n");
    outputWindowPane.OutputString(codeElement.GetEndPoint().Line + "\n");
    outputWindowPane.OutputString(codeElement.FullName + "\n");
    outputWindowPane.OutputString(codeElement.Parameters.Count + "\n");
    // Itemの列挙ができない模様
    /*
    for (var i = 0; i < codeElement.Parameters.Count; ++i) {
        // Itemが認識できない
        outputWindowPane.OutputString(codeElement.Parameters.Item(i) + "\n");
    }
    */
    outputWindowPane.OutputString(codeElement.DocComment + "\n");
    
}
```  
  
## 時刻のインサート  
  
```js
/// <reference path="C:\Users\ユーザー名\AppData\Local\Microsoft\VisualStudio\16.0_ea48cace\Macros\dte.js" />
var date = new Date();

var day = date.getDate();
var month = date.getMonth() + 1;
var year = date.getYear();

var hours = date.getHours();
var minutes = date.getMinutes();

// Add a zero if single digit
if (day <= 9) day = "0" + day;
if (month <= 9) month = "0" + month;
if (minutes <= 9) minutes = "0" + minutes;
if (hours <= 9) hours = "0" + hours;

Macro.InsertText("//" + year + "/" + month + "/" + day + ", " + hours + ":" + minutes);
```  
  
  
# まとめ  
VisualStudioのマクロは復活しましたが、デバッグ機能がない、すべての機能が使えるわけでもない、メンテナンスする気はないとうたっているので、使いどころは限られるようです。  
