# 概要  
Visual Studio のProfessionalEdition以上ではマクロを使用することができる。  
そのマクロを使用することで、VisualStudioの操作を自動化することが可能である。  
今回はVCのコードでDoxygenコメントを記述するマクロを作成する。  
  
VisualStudio2008 Proffesionalでサンプルを記述しているが、~~新しいバージョンでも同等のことができる。~~  
  
Vs2012から使えないの拡張機能を使う必要がある。  
  
http://variantsoft.co.jp/blog/2014/01/14/visualstudio2013%E3%81%AE%E6%8B%A1%E5%BC%B5%E6%A9%9F%E8%83%BD%E3%82%92%E4%BD%9C%E3%82%8B/  
  
(2019/12/15 追記）  
**復活のVisualStudioのマクロ**  
https://github.com/mima3/note/blob/master/復活のVisualStudioのマクロ.md  
いくつかの制限付きながら拡張機能としてVisualStudioのマクロが復活しているようです。  
  
  
  
# マクロの登録方法  
１．[ツール]→[マクロ]→[マクロエクスプローラ]を選択する。  
![macro.png](/image/f783cf4e-ebc9-999e-d87b-c05a33f37fc9.png)  
  
２．マクロエクスプローラにて新規にモジュールを追加する。  
![macro.png](/image/464069d1-19f8-469a-4969-69d5b56ca769.png)  
  
３．モジュール追加のダイアログで任意の名前でモジュールを追加する。  
![macro.png](/image/1bda2060-7fca-7763-1890-8402f3b74c96.png)  
  
４．マクロエクスプローラで追加したモジュールを選択して編集を行う  
![macro.png](/image/b8a89810-e687-9023-c2da-58180c90b624.png)  
  
５．MicrosoftVisual Studio Macrosが起動するので以後、そこでマクロを記述する。  
  
![macro.png](/image/01daae6d-7d8e-8b21-df6d-c27cbadf5307.png)  
  
# 選択中の項目にDoxygenコメントを記述するコード  
  
```vb

Imports System
Imports EnvDTE
Imports EnvDTE80
Imports EnvDTE90
Imports System.Diagnostics

Public Module mdlVCSrcMacros
    ' カーソルで選択中の関数/Class/EnumにDoxgenで使用するコメントを付与します.
    ' 何も選択せずにファイルの先頭にある場合⇒ファイル用のヘッダコメント
    ' cppファイルで関数を選択して実行⇒関数用ヘッダ
    ' ※クラスのメンバはClassViewが認識していないとコメントを付与してくれない
    '   TODO:課題
    ' classを選択して実行⇒クラス用ヘッダ
    ' enumを選択して実行⇒enum用ヘッダ.
    ' 参考
    '  http://www.microsoft.com/japan/msdn/academic/Articles/fun/06/
    Sub CreateHeaderComment()
        Dim sel As TextSelection = DTE.ActiveDocument.Selection
        Dim strCom As String
        Dim myFCM As FileCodeModel = DTE.ActiveDocument.ProjectItem.FileCodeModel
        strCom = ""

        ' ファイルヘッダ.
        If sel.TopPoint.Line = 1 And sel.Text.Length = 0 Then
            sel.Text = "/**"
            sel.NewLine()
            sel.Text = " * @file" + vbTab + DTE.ActiveDocument.Name
            sel.NewLine()
            sel.Text = "* @brief" + vbTab + "Copyright(c) " + Year(Today()).ToString()
            sel.Text = vbTab + "Nikon Corporation - All rights reserved."
            sel.NewLine()
            sel.Text = "* @date" + vbTab + Format(Today(), "yyyy/MM/dd")
            sel.NewLine()
            sel.Text = "* @author" + vbTab + "作者名"
            sel.NewLine()
            sel.Text = "* @note" + vbTab + ""
            sel.NewLine()
            sel.Text = "*/"
            Exit Sub
        End If
        Dim objCodeElement As CodeElement ' コード情報取得用オブジェクト

        ' 関数ヘッダ
        objCodeElement = myFCM.CodeElementFromPoint(sel.ActivePoint, vsCMElement.vsCMElementFunction) ' ソースコード情報の取得
        If IsNothing(objCodeElement) = False Then
            If objCodeElement.Kind = vsCMElement.vsCMElementFunction Then
                ' 関数の場合.
                Dim codeFun As CodeFunction = objCodeElement
                strCom = "/**" + vbCrLf
                strCom = strCom + " * " + codeFun.Name + vbCrLf
                strCom = strCom + "* " + vbCrLf
                Dim j As Integer
                For j = 1 To codeFun.Parameters.Count
                    Dim codeParam As CodeParameter
                    codeParam = codeFun.Parameters.Item(j)
                    strCom = strCom + "* @param[in/out]" + vbTab + codeParam.Type.AsString + vbTab + codeParam.Name + vbCrLf
                Next j
                If codeFun.Parameters.Count > 1 Then
                    strCom = strCom + "* " + vbCrLf
                End If
                strCom = strCom + "* @return" + vbTab + codeFun.Type.AsString + vbCrLf
                strCom = strCom + "* @note" + vbCrLf
                strCom = strCom + "*/" + vbCrLf
                sel.MoveToPoint(objCodeElement.StartPoint)
            End If
        End If

        ' class 名
        objCodeElement = myFCM.CodeElementFromPoint(sel.ActivePoint, vsCMElement.vsCMElementClass) ' ソースコード情報の取得
        If IsNothing(objCodeElement) = False Then
            If objCodeElement.Kind = vsCMElement.vsCMElementClass Then
                strCom = "/**" + vbCrLf
                strCom = strCom + " * @class" + vbTab + objCodeElement.Name + vbCrLf
                strCom = strCom + "* @brief" + vbCrLf
                strCom = strCom + "*/" + vbCrLf
                sel.MoveToPoint(objCodeElement.StartPoint)
            End If

        End If

        ' enum 名
        objCodeElement = myFCM.CodeElementFromPoint(sel.ActivePoint, vsCMElement.vsCMElementEnum) ' ソースコード情報の取得
        If IsNothing(objCodeElement) = False Then
            If objCodeElement.Kind = vsCMElement.vsCMElementEnum Then
                strCom = "/**" + vbCrLf
                strCom = strCom + " * @enum" + vbTab + objCodeElement.Name + vbCrLf
                strCom = strCom + "* @brief" + vbCrLf
                strCom = strCom + "*/" + vbCrLf
                sel.MoveToPoint(objCodeElement.StartPoint)
            End If

        End If

        ' struct
        objCodeElement = myFCM.CodeElementFromPoint(sel.ActivePoint, vsCMElement.vsCMElementStruct) ' ソースコード情報の取得
        If IsNothing(objCodeElement) = False Then
            If objCodeElement.Kind = vsCMElement.vsCMElementStruct Then
                Dim codeStruct As CodeStruct = objCodeElement

                strCom = "/**" + vbCrLf
                strCom = strCom + " * @struct" + vbTab + objCodeElement.Name + vbCrLf
                strCom = strCom + "* @brief" + vbCrLf
                strCom = strCom + "*/" + vbCrLf
                sel.MoveToPoint(objCodeElement.StartPoint)
            End If
        End If


        If Not strCom.Length = 0 Then
            sel.StartOfLine(vsStartOfLineOptions.vsStartOfLineOptionsFirstColumn)
            sel.LineUp(True)
            sel.Text = strCom
        End If

    End Sub

End Module
```  
  
DTEインターフェイスを経由することで、VisualStudioの操作がおこなえる。  
http://msdn.microsoft.com/ja-jp/library/envdte.dte.aspx  
  
## 現在選択中のコードの要素を列挙  
  
```vb

        ''' ソース ファイル内のコード要素または構成体を表します
        Dim myFCM As FileCodeModel = DTE.ActiveDocument.ProjectItem.FileCodeModel
        Dim objCodeElement As CodeElement
        For Each objCodeElement In myFCM.CodeElements
            MsgBox(objCodeElement.FullName & ":" & objCodeElement.Kind)
        Next
```  
  
CodeElement インターフェイス  
http://msdn.microsoft.com/ja-jp/library/envdte.codeelement.aspx  
  
kindを見ることで、Inculdeなのか、関数なのか、クラスなのか判断することができる。  
  
## カーソルの座標とKindからコードの要素を取得  
  
```vb

Dim myFCM As FileCodeModel = DTE.ActiveDocument.ProjectItem.FileCodeModel
Dim objCodeElement As CodeElement ' コード情報取得用オブジェクト
objCodeElement = myFCM.CodeElementFromPoint(sel.ActivePoint, vsCMElement.vsCMElementFunction) ' ソースコード情報の取得
```  
  
この関数はカーソルの位置とKindの種類をもとにしてコード要素を取得する。  
条件があわなければnullとなる。  
  
## カーソルの位置を関数の先頭にあわせて文字を記述する  
  
```vb

Dim sel As TextSelection = DTE.ActiveDocument.Selection

'コード要素の先頭に移動
sel.MoveToPoint(objCodeElement.StartPoint)

'一列目に移動
sel.StartOfLine(vsStartOfLineOptions.vsStartOfLineOptionsFirstColumn)

' 一行うえへ
sel.LineUp(True)

' 文字を記録
sel.Text = "XXXXXXXXXXXXXXX"

```  
  
# マクロの実行方法  
１．ソースコードの任意の箇所を選択。  
  
![macro.png](/image/2518d47f-6b7b-1ab1-f08e-b69021419e30.png)  
  
２．マクロエクスプローラの「CreateHeaderComment()」をダブルクリックする。  
３．マクロが実行されると以下のようにDoxygenコメントが付与される。  
![macro.png](/image/bb74bd03-27e1-0c6b-e321-489e02de85ac.png)  
  
## ショートカットの登録  
作成したマクロはショートカットキーに割り当てることができる。  
  
１． [ツール] メニューの [オプション] をクリックして、[オプション] ダイアログ ボックスを表示する。  
２．[環境] フォルダーの [キーボード] をクリックする。  
３．[以下の文字列を含むコマンドを表示] ボックスにマクロ名を入力して絞りこみ、選択する。  
![macro.png](/image/18501c4b-6c9f-4dfb-61ed-2325a16f63cc.png)  
  
４．[ショートカット キー] ボックスをクリックし、特定のキーの組み合わせ押し「割り当て」ボタンをクリックする  
  
以降設定したキーの組み合わせで、指定のマクロが動作する。  
  
 __方法: マクロを実行する__   
http://msdn.microsoft.com/ja-jp/library/vstudio/a0003t62%28v=vs.100%29.aspx  
  
# 応用  
この記事でVisualStudioの自動化が行えることを説明した。  
  
VisualStudioのマクロを使う例としては以下のようなことが考えられる。  
・あるルールに従って、自動でコードを作成する。  
・プロジェクトの設定を自動で行う  
・プロジェクトエクスプローラーからファイルを選択してバージョン管理ソフトにコミットしたり、元に戻したりする。  
  
  
  
