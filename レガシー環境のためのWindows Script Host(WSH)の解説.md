ここでは、XPやVista以前のレガシーな環境でのスクリプト処理を行うためのWSHについて説明します。  
  
 __もし、Windows7以降のみが前提であれば、Windows PowerShell の使用も検討しましょう。__   
→*ただしPowerShellだと[OfficeのCOMの解放処理が面倒だったり](https://qiita.com/mima_ita/items/aa811423d8c4410eca71)、[InternetExploreの操作が安定しなかったりする](https://qiita.com/mima_ita/items/4149a4cdb9a33084258b#powershell%E3%81%A7%E3%81%AEie%E6%93%8D%E4%BD%9C%E3%81%AE%E5%95%8F%E9%A1%8C%E7%82%B9)*  
  
# 概要  
WSHとはWindows Script Hostの略です。  
  
Microsoft Visual Basic Scripting Edition (以下 VBScript) と, Microsoft JScriptの２種類がサポートされています。※  
※環境によってはEdgeのエンジンが使用可能です。[参照](https://qiita.com/mima_ita/items/453bb6c313e459c44689#comment-c1ebd3350d93efe9d8cf)  
  
JScriptはMS独自の規格で一般なJavaScriptとは規格が異なります。  
  
最大の強みは開発環境のないマシンでも修正、実行が行えることで、バッチファイルより複雑な処理を実現することが可能です。  
  
具体的には下記のようなことが可能です。   
  
  
・システム情報の取得と変更  
・ファイル・フォルダの作成、コピー、属性変更  
・レジストリの読み書き  
・プログラムの実行  
・COMコンポーネントの作成・利用  
  
  
  
# スクリプトの作成と実行  
ここではスクリプトの作成方法と実行方法について記述します。  
まず、下記のようなファイルを作成してください。   
  
```Hello.js
var sHello;
sHello="Hello World!";
WScript.Echo(sHello);
```  
このプログラムは"HelloWorld!"の文字を表示するだけのプログラムです。 このプログラムをWSHで実行させるには下記のコマンドをコマンドプロンプトから実行してください。   
  
```
CScript Hello.js
```  
  
コマンドプロンプトに文字が出力されたと思います。  
  
実は、WSHは上記の例のようなコマンドラインベースのものだけでなく、GUIを使用した対話型のプログラムも作成できます。  
  
コマンドプロンプトで下記のようなコマンドを実行してください。   
  
```
WScript Hello.js
```  
  
おそらく、メッセージボックスが表示されたかと思います。  
また、CScriptとWScriptは32bit用と64bit用のそれぞれが存在します。  
  
|Path|説明|  
|:---|:---|  
|C:\Windows\SysWOW64\CScript.exe|32ビット用 コンソール表示に対応したプログラム|  
|C:\Windows\SysWOW64\WScript.exe|32ビット用 ウィンドウに対応したプログラムに対応したプログラム|  
|C:\Windows\system32\CScript.exe|64ビット用 コンソール表示に対応したプログラム|  
|C:\Windows\system32\WScript.exe|64ビット用 ウィンドウに対応したプログラムに対応したプログラム|  
  
COMを使用する場合、64bitで動かすか、32bitで動かすかは重要なことになります。  
  
## CScriptおよびWScriptのオプション  
|パラメータ|説明|  
|:---------|:---|  
|//B |バッチ モードでスクリプトを実行します。ユーザー プロンプトおよびスクリプト エラーをコマンド ライン表示しません。既定の設定は、対話モードです。|  
|//D |アクティブ デバッグを使用可能にします。|  
|//E:engine |指定したスクリプト エンジンでスクリプトを実行します。|  
|//H:CScript |または //H:Wscript CScript.exe または WScript.exe をスクリプト実行用の既定アプリケーションとして登録します。どちらも指定しなければ、WScript.exe が既定のアプリケーションとみなされます。|  
|//I |既定値です。対話モードでスクリプトを実行します。バッチ モードとは逆に、ユーザー プロンプトおよびスクリプト エラーの表示を有効にします。|  
|//Job:<JobID> |指定した JobID を .wsf ファイルから実行します。|  
|//logo |既定値です。バナーを表示します。nologo の逆の設定です。|  
|//nologo |実行時にバナーを表示しません。既定の設定では、logo が適用されます。|  
|//S |このユーザーに対して、現在のコマンド ライン オプションを保存します。|  
|//T:nn |スクリプトの実行を継続できる時間の上限 (タイムアウト) を設定します。既定では、スクリプトは制限なしで実行されます。//T パラメータでタイムアウトを設定すると、スクリプトが長時間にわたって実行されるのを防止できます。実行時間が指定値を超過すると、CScript が IActiveScript::InterruptThread メソッドを使用してスクリプト エンジンに割り込み、プロセスを強制終了します。|  
|//U |Windows NT および Windows 2000 で使用できるオプションです。コマンド ライン出力を Unicode にします。CScript には、Unicode と ANSI を自動的に判別する機能はありません。既定の設定では、ANSI が使用されます。|  
|//X |スクリプトをデバッガで実行します。|  
|//? |コマンド パラメータの説明および使用法に関する概略を表示します|  
http://msdn.microsoft.com/ja-jp/library/cc392525.aspx  
  
# デバッグ方法  
WSHはVisual Studioでデバッグを行うことが可能です。  
Visual StudioはExpressの場合にデバッグが行えないかもしれません。Professional以降が必要かもしれません。  
  
  
  
１．//xオプションを付与してWSHを実行します  
  
```
CScript Hello.js //x
```  
  
２．下記のようなウィンドウが表示されるのでデバッグを行いたいVisualStudioのバージョンを選択して「はい」を押してください。  
![無題.png](/image/57318b41-20d5-61cf-1654-6de05f9e6b4d.png)  
  
３．あとは通常のVSのデバッグが使用できます。  
![無題.png](/image/d79c4c9d-4fa9-6a6c-7a00-b69b9208fa0f.png)  
  
# 外部のファイルを利用する方法  
  
以下の例では外部に記述したVBScriptを使用する方法について記述します。  
  
**.\lib\ADOCtrl.vbs**  
```vb:.\lib\ADOCtrl.vbs
Option Explicit

'*
'* ADOCtrlのオブジェクトを作成・取得する.
'* @return ADOCtrl
'*
Function getADOCtrl()
    set getADOCtrl = new ADOCtrl
End Function

'! @class ADOCtrl
'!
class ADOCtrl
    Private m_objADOCnn
    
    '* DBに接続
    '* @param[in] host ホスト名
    '* @param[in] db   データベース名
    '* @param[in] user ユーザ名
    '* @param[in] pass パスワード
    '*
    Public Function ConnectSqlServer(Byval host, Byval db, Byval user, Byval pass )
        Set m_objADOCnn = CreateObject("ADODB.Connection")
        
        Dim cnn
        
        cnn = "PROVIDER=SQLOLEDB" & _
              ";SERVER=" & host & _
              ";DATABASE=" & db & _
              ";UID=" & user & _ 
              ";PWD=" & pass & ";"
        
        Call m_objADOCnn.Open( cnn )
        If m_objADOCnn.Errors.Count = 0 Then
            ConnectSqlServer = True
        Else
            ConnectSqlServer = False
        End If
    End Function
    
    
    '*
    '* DBの接続を切断する。
    '*
    Public Sub Close
        If Not m_objADOCnn Is Nothing Then
            m_objADOCnn.Close
            Set m_objADOCnn = Nothing
        End If
    End Sub
    
    '* 
    '* SQLの実行
    '* @param[in]  SQL文
    '* @param[out] 出力結果
    '*
    Public Function ExcuteSQL( ByVal sSQL, ByRef outRet )
        Dim objRS
        ExcuteSQL = False
        
        Set objRS = CreateObject("ADODB.Recordset")
        Call objRS.Open( sSQL, m_objADOCnn, 0, 1, 1 )
        If m_objADOCnn.Errors.Count > 0 Then
            Set objRS = NoThing
            Exit Function
        End If

        If objRS.EOF Then
            Set objRS = NoThing
            Exit Function
        End If
        
        outRet = objRS.GetRows()
        objRS.Close
        
        Set objRS = NoThing
        ExcuteSQL = True
        
    End Function
End Class
```  
  
外部ファイルの使用例  
  
```xml


<?XML version="1.0" encoding="Shift_JIS" ?>
<package>
<job id="GetTSQLFunction()">
<comment>
ADOCtrlのテスト
</comment>
<script language="VBScript" src=".\lib\ADOCtrl.vbs"/>
<script language="VBScript">
<![CDATA[
        Option Explicit
        
        Dim DBCtrl
        Set DBCtrl = getADOCtrl()
        
        IF Not DBCtrl.ConnectSqlServer("XXX-PC","TestDB","username","password") Then
                WScript.Quit -1
        End If
        
        Dim vRecs       ' 列,行
        If Not DBCtrl.ExcuteSQL("SELECT    specific_name,object_definition(object_id(specific_name)) FROM  information_schema.routines ORDER BY specific_name",vRecs) Then
                WScript.Quit -1
        End If
        Dim i
        Dim objData
        For i = LBound(vRecs,2) To UBound(vRecs,2)
                Call WScript.Echo("=======================================") 
                Call WScript.Echo(vRecs(0,i) )
                Call WScript.Echo( vRecs(1,i)) 
        Next
        Call DBCtrl.Close()
        Set DBCtrl = Nothing

]]>
</script>
</job>
</package>
```  
  
以下のようにscriptタグのsrcを付与することで外部ファイルを使用できるようになります。  
  
```
<script language="VBScript" src=".\lib\ADOCtrl.vbs"/>
```  
