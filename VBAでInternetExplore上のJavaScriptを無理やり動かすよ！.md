みんな大好きInternetExplore11のJavaScriptを、みんな大好きなExcelのVBAから動かします。  
  
## なにが嬉しいの？  
・現在、表示されているIEのJavaScriptの保持している変数の中身を出力できるよ！  
・無理やりJavaScriptで例外を発生させたときの挙動を確かめられるよ！  
・サーバー側のソースコードを書き換えずにローカル環境でJavaScriptの関数をテスト用に置き換えられるよ！  
  
ExcelVBAなので、手足を縛ってプログラムを開発するのが性癖なマゾい会社でも使えるよ！やったぜ！  
  
## 環境  
InternetExploere  
Windows10  
Office16 Excel 32bit  
  
なお、64bitのExcelを使っている箇所はlongでなく、longPtrとかにしないと動かないと思うけど、ぶっちゃけ64bitマシン支給するようなブルジョワジーの組織だったらVisualStudio使わせてもらってC#で書いた方がいいと思います。  
  
## サンプル  
  
**IEAuto.bas**  
```vb:IEAuto.bas
 Option Explicit

' Microsoft HTML Object Libraryを参照設定すること

Private Declare Function EnumWindows Lib "user32" (ByVal lpEnumFunc As Long, ByVal lParam As Long) As Long
Private Declare Function EnumChildWindows Lib "user32" (ByVal hWndParent As Long, ByVal lpEnumFunc As Long, ByVal lParam As Long) As Long

Private Declare Function GetClassName Lib "user32" Alias "GetClassNameA" (ByVal hwnd As Long, ByVal lpClassName As String, ByVal nMaxCount As Long) As Long
Private Declare Function RegisterWindowMessage Lib "user32" Alias "RegisterWindowMessageA" (ByVal lpString As String) As Long
Private Declare Function SendMessageTimeout Lib "user32" Alias "SendMessageTimeoutA" (ByVal hwnd As Long, ByVal msg As Long, ByVal wParam As Long, ByVal lParam As Long, ByVal fuFlags As Long, ByVal uTimeout As Long, lpdwResult As Long) As Long
Private Declare Function ObjectFromLresult Lib "oleacc" (ByVal lResult As Long, riid As Any, ByVal wParam As Long, ppvObject As Object) As Long

Private mIEWndList As Collection
Private Type UUID
  Data1 As Long
  Data2 As Integer
  Data3 As Integer
  Data4(0 To 7) As Byte
End Type

' MSHTML.IHTMLDocumentのキャッシュを検索する
Public Sub RefreshBrowserCache()
    Set mIEWndList = New Collection
    
    Call EnumWindows(AddressOf EnumIEWndProc, 0)
End Sub
' 指定の名称を含むタイトルのブラウザを検索する
Public Function FindBrowser(ByVal title As String) As MSHTML.IHTMLDocument2
    Dim r As MSHTML.IHTMLDocument2
    
    If mIEWndList Is Nothing Then
        Call RefreshBrowserCache
    End If
    
    Set r = FindBrowserInCache(title)
    If Not r Is Nothing Then
        Set FindBrowser = r
        Exit Function
    End If
    
    Call RefreshBrowserCache
    Set FindBrowser = FindBrowserInCache(title)
    
End Function

' キャッシュの中から指定の名称を含むタイトルのブラウザを検索する
Private Function FindBrowserInCache(ByVal title As String) As MSHTML.IHTMLDocument2
    Dim d As MSHTML.IHTMLDocument2
    For Each d In mIEWndList
        If InStr(d.title, title) > 0 Then
            Set FindBrowserInCache = d
            Exit Function
        End If
    Next
    
End Function



' コールバック関数
Private Function EnumIEWndProc(ByVal hwnd As Long, lParam As Long) As Boolean
    Const FRAME_NAME = "IEFrame"
    Const FRAME_NAME_EDGE = "TabWindowClass" 'Edgeの場合のクラス名
    
    Dim strClassName As String * 128
    Dim lngRet As Long
    
    strClassName = ""
    Call GetClassName(hwnd, strClassName, 128)
    Dim c As String
    c = Left(strClassName, Len(Trim(strClassName)) - 1)
    If c = FRAME_NAME Or c = FRAME_NAME_EDGE Then
        EnumChildWindows hwnd, AddressOf EnumIEServerWndProc, 0
    End If
    'Debug.Print "[" & Trim(strClassName) & "_]"
    EnumIEWndProc = True
End Function

' コールバック関数
Private Function EnumIEServerWndProc(ByVal hwnd As Long, ByVal lParam As Object) As Long
    Const SERVER_NAME = "Internet Explorer_Server"
    Dim strClassName As String * 128
    Dim lngRet As Long
    Dim doc As MSHTML.IHTMLDocument2
    
    strClassName = ""
    Call GetClassName(hwnd, strClassName, Len(strClassName))
    If Left(strClassName, Len(Trim(strClassName)) - 1) = SERVER_NAME Then
        Set doc = GetHTMLDocument(hwnd)
        If Not doc Is Nothing Then
            mIEWndList.Add doc
        End If
    End If
    EnumIEServerWndProc = 1
End Function

Private Function GetHTMLDocument(ByVal hwnd As Long) As MSHTML.IHTMLDocument2
    Dim doc As MSHTML.IHTMLDocument2
    Dim lMsg As Long
    Dim lRet As Long
    Dim hr As Long
    Dim IID_IHTMLDocument As UUID
    
    With IID_IHTMLDocument
        .Data1 = &H626FC520
        .Data2 = &HA41E
        .Data3 = &H11CF
        .Data4(0) = &HA7
        .Data4(1) = &H31
        .Data4(2) = &H0
        .Data4(3) = &HA0
        .Data4(4) = &HC9
        .Data4(5) = &H8
        .Data4(6) = &H26
        .Data4(7) = &H37
    End With
    lMsg = RegisterWindowMessage("WM_HTML_GETOBJECT")
    If lMsg <> 0 Then
        SendMessageTimeout hwnd, lMsg, 0, 0, &H2, 1000, lRet
        If lRet <> 0 Then
            If ObjectFromLresult(lRet, IID_IHTMLDocument, 0, doc) = 0 Then
                Set GetHTMLDocument = doc
            End If
        End If
    End If
End Function

```  
  
**サンプル**  
```vb:サンプル
Public Sub test()
    Dim b As MSHTML.IHTMLDocument3
    Dim w As Object

    Set b = IEAuto.FindBrowser("Google")
    If b Is Nothing Then
        Debug.Print "見つからない"
        Exit Sub
    End If
    
    Dim r As Variant
    Dim elems As MSHTML.IHTMLElementCollection
    Dim elem As MSHTML.IHTMLElement
    Set elems = b.getElementsByName("btnK")
    Debug.Print elems.Item(0).getAttribute("value")
    
    ' 以下のような感じでJavaScriptにアクセスできるがEdgeの場合はアクセス違反になる
    ' 管理者権限も無駄
    Dim testId As Variant
    testId = Timer
    b.parentWindow.execScript ("function x() { result = 5+5+4; console.log('TEST'); var d = document.createElement('div'); d.id = 'test_elem" & testId & "'; d.innerHTML  = result; document.getElementsByTagName('body').item(0).appendChild(d); }; x();")
     
    Set elem = b.getElementById("test_elem" & testId)
    Debug.Print elem.innerHTML
    
End Sub
```  
  
## 解説  
IEのウィンドウハンドルからMSHTML.IHTMLDocumentを取得してあとはよくあるIEの自動操作をしています。  
IHTMLDocumentに変換できるウィンドウのクラス名は「Internet Explorer_Server」です。  
IE11の場合は「IEFrame」クラスの子ウィンドウになっています。  
Edgeの場合は「TabWindowClass」クラスの子ウィンドウになっています。  
  
プログラムとしては「IEAuto.FindBrowser("Google")」でタイトルにGoogleを含むものを検索してIHTMLDoucmentを取得します。  
あとは、exeScriptでJavaScriptをたたくことが可能ですが、関数の仕様として戻り値を取得することはできません。  
  
そのため、JavaScriptから値をVBAに返す場合は、テスト用の要素に書き込むといいでしょう。  
  
なお、すくなくとも当方の環境だとexeScriptはEdgeだとアクセス違反になりました。  
  
## 参考  
Cant create HTML document from Hwnd using C#  
https://stackoverflow.com/questions/20873885/cant-create-html-document-from-hwnd-using-c-sharp  
  
【2017年1月版】Microsoft Edgeを操作するVBAマクロ(DOM編)  
https://www.ka-net.org/blog/?p=7921  
  
  
  
