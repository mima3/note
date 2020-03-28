# 背景  
JsLint for VisualStudioはJavaScriptの静的解析を行うjslintやjshintをVisualStudio上で動かすVisualStudio用のアドインである。  
  
https://jslint4vs2010.codeplex.com/  
  
しかしながら、2012年が最後のリリースでメンテナンスをしていない。  
そのため、以下のようなコードはエラーとなり、オプションで回避もできない。  
  
```js
switch (1) {
    case 1:
        x = 3;
        break;
    case 2:
        x = 4;
        break;
    case 3:
        x = 5;
        break;
    default:
        x = 3;
        break;
}

/* 改造前のただしい書き方
switch (1) {
case 1:
    x = 3;
    break;
case 2:
    x = 4;
    break;
case 3:
    x = 5;
    break;
default:
    x = 3;
    break;
}*/

```  
  
https://jslint4vs2010.codeplex.com/workitem/1446  
  
このドキュメントではこれを何とかするものである。  
  
# JSLint.VS2012.vsixの中身  
VSIXはVisualStudioの拡張配置である。このVSIXはzipファイルにすぎない。  
拡張子をzipにしてみよう。  
  
次のような内容が確認できる。  
  
![jslint1.png](/image/ac8f7237-3342-934e-168c-2dae2f4f61f3.png)  
  
JSLint.Framework.dllが実際のjslintなどの処理を行っているプログラムである。  
このDLLはjavascriptファイルをリソースとしてもっており、そのJavaScriptをNoesis.Javascript.dllを利用して実行し、静的解析を行っている。  
  
# JSLint.Framework.dllの改造方法  
## JSLint.Framework.dllの使用方法  
JSLint.Framework.dllを参照して以下のようなプログラムを実行する。  
  
```csharp
using System;
using System.Collections.Generic;
using JSLint.VS2010.LinterBridge;
using JSLint.VS2010.OptionClasses;
namespace ConsoleApplication1
{
    class Program
    {
        static void Main(string[] args)
        {
            var jslint = new JSLint.VS2010.LinterBridge.JSLinter();
            var option = new JSLintOptions();
            option = JSLintOptions.Default;
            option.SelectedLinter = Linters.JSHint;
            string script = @"
var x = 9;
switch (1) {
    case 1:
        x = 3;
        break;
    case 2:
        x = 4;
        break;
    case 3:
        x = 5;
        break;
    default:
        x = 3;
        break;
}
/* 改造前のただしい書き方
switch (1) {
case 1:
    x = 3;
    break;
case 2:
    x = 4;
    break;
case 3:
    x = 5;
    break;
default:
    x = 3;
    break;
}*/
x = x * 2;
";
            List<JSLintError> list = jslint.Lint(script, option, true);
            foreach (var x in list)
            {
                Console.WriteLine(x.Line + ":" + x.Message);
            }
        }
    }
}

```  
  
おそらく、jshintでエラーが発生するだろうが、これをなんとかすればよい。  
  
## .NETのDLLの埋め込みリソースを取得する  
VisualStudioの開発者コマンド プロンプトから以下のコマンドを実行すると、埋め込みリソースが展開される。  
  
```
ildasm ../JSLint.Framework.dll /out=JSLint.Framework.il
```  
  
![jslint2.png](/image/8a21202a-960e-960a-2168-75047d26e3df.png)  
  
## jslintの比較方法を修正する。  
JSLint.Framework.JS.jshint.jsを以下のように修正する。07-25-2015でコメントを打っている箇所が修正箇所だ。  
  
```js
    blockstmt("switch", function () {
        var t = nexttoken,
            g = false;
        funct["(breakage)"] += 1;
        advance("(");
        nonadjacent(this, t);
        nospace();
        this.condition = expression(20);
        advance(")", t);
        nospace(prevtoken, token);
        nonadjacent(token, nexttoken);
        t = nexttoken;
        advance("{");
        nonadjacent(token, nexttoken);
        indent += option.indent;
        this.cases = [];
        for (;;) {
            switch (nexttoken.id) {
            case "case":
                switch (funct["(verb)"]) {
                case "break":
                case "case":
                case "continue":
                case "return":
                case "switch":
                case "throw":
                    break;
                default:
                    // You can tell JSHint that you don't use break intentionally by
                    // adding a comment /* falls through */ on a line just before
                    // the next `case`.
                    if (!ft.test(lines[nexttoken.line - 2])) {
                        warning(
                            "Expected a 'break' statement before 'case'.",
                            token);
                    }
                }
                // indentation(-option.indent); m.ita 07-25-2015 for ignore switch indent.
                advance("case");
                this.cases.push(expression(20));
                increaseComplexityCount();
                g = true;
                advance(":");
                funct["(verb)"] = "case";
                break;
            case "default":
                switch (funct["(verb)"]) {
                case "break":
                case "continue":
                case "return":
                case "throw":
                    break;
                default:
                    if (!ft.test(lines[nexttoken.line - 2])) {
                        warning(
                            "Expected a 'break' statement before 'default'.",
                            token);
                    }
                }
                //indentation(-option.indent);  m.ita 07-25-2015 for ignore switch indent.
                advance("default");
                g = true;
                advance(":");
                break;
            case "}":
                indent -= option.indent;
                indentation();
                advance("}", t);
                if (this.cases.length === 1 || this.condition.id === "true" ||
                        this.condition.id === "false") {
                    if (!option.onecase)
                        warning("This 'switch' should be an 'if'.", this);
                }
                funct["(breakage)"] -= 1;
                funct["(verb)"] = undefined;
                return;
            case "(end)":
                error("Missing '{a}'.", nexttoken, "}");
                return;
            default:
                indent += option.indent;  //m.ita 07-25-2015 for ignore switch indent.
                if (g) {
                    switch (token.id) {
                    case ",":
                        error("Each value should have its own case label.");
                        return;
                    case ":":
                        g = false;
                        statements();
                        break;
                    default:
                        error("Missing ':' on a case clause.", token);
                        return;
                    }
                } else {
                    if (token.id === ":") {
                        advance(":");
                        error("Unexpected '{a}'.", token, ":");
                        statements();
                    } else {
                        error("Expected '{a}' and instead saw '{b}'.",
                            nexttoken, "case", nexttoken.value);
                        return;
                    }
                }
                indent -= option.indent;  //m.ita 07-25-2015 for ignore switch indent.
            }
        }
    }).labelled = true;
```  
  
## .NETの埋め込みリソースを変更してDLLを作り直す  
JavaScriptを修正したら、以下のコマンドでDLLを作り直す。  
  
```
ilasm JSLint.Framework.il /dll
```  
  
## 動作確認  
JSLint.Framework.dllを使用したプログラムを再実行すれば、switchのインデントでjslintが文句をいわなくなったことが確認できる。  
  
  
## 参考  
 **Is it possible to Add/Remove/Change an embedded resource in .NET DLL?**   
http://stackoverflow.com/questions/6545858/is-it-possible-to-add-remove-change-an-embedded-resource-in-net-dll  
  
  
## ひとこと  
自分でメンテできなかったり、金でメンテさせることができないツールを安易に採用してはいけない。  
  
なお、お助け料一億万円でいいぞ！ローンも可。  
