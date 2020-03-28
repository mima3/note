このドキュメントではWinMergeを用いてSQLiteの比較を行う方法について解説する。  
  
# 導入方法  
１．SQLite用のODBCをインストールする。  
http://www.ch-werner.de/sqliteodbc/  
  
下記のいずれか、または両方インストールすること。  
  
sqliteodbc.exe  
sqliteodbc_w64.exe   
  
２．次のようなファイルを作成する  
  
**SqliteToText.sct**  
```vbnet:SqliteToText.sct
<scriptlet>

<implements type="Automation" id="dispatcher">
	<property name="PluginEvent">
	          <get/>
        </property>
	<property name="PluginDescription">
	          <get/>
        </property>
	<property name="PluginFileFilters">
	          <get/>
        </property>
	<property name="PluginIsAutomatic">
	          <get/>
        </property>
      	<method name="UnpackFile"/>
      	<method name="PackFile"/>
</implements>

<script language="VBS">
Option Explicit

Function get_PluginEvent()
         get_PluginEvent = "FILE_PACK_UNPACK"
End Function

Function get_PluginDescription()
         get_PluginDescription = "SqliteToText"
End Function

Function get_PluginFileFilters()
         get_PluginFileFilters = "\.sqlite(\..*)?$;\.sqlite3(\..*)?$;\.db(\..*)?$"
End Function

Function get_PluginIsAutomatic()
         get_PluginIsAutomatic = True
End Function

Function UnpackFile(fileSrc, fileDst, pbChanged, pSubcode)
	Dim cnn
	Dim Filename
	Dim rs
	Dim i
	Dim tableNameDict
	Dim name
	Dim fso
	Dim fo

	Set fso = CreateObject("Scripting.FileSystemObject")
	Set fo = fso.CreateTextFile(fileDst, True)

	Set cnn = CreateObject("ADODB.Connection")
	cnn.CursorLocation = 3 '
	FileName = "test.sqlite"
	cnn.Open "DRIVER=SQLite3 ODBC Driver;Database=" & fileSrc & ";"
		
	Set rs = cnn.Execute("SELECT * FROM sqlite_master;")
	i = 0

	rs.MoveFirst
	Do While Not rs.EOF
		name = rs.Fields("Name").Value
		fo.WriteLine "[" & name & "]"
		fo.WriteLine rs.Fields("sql").Value
		If rs.Fields("Type").Value = "table" Then
			call showTable(fo, cnn , name)
		End if
		rs.MoveNext
		i = i + 1
	Loop 

	rs.Close
	Set rs = Nothing


	cnn.Close
	Set cnn = Nothing
	
	fo.Close
	Set fo = Nothing
	Set fso = Nothing
	
	pbChanged = True
	pSubcode = 0
	UnpackFile = True

End Function

Function PackFile(fileSrc, fileDst, pbChanged, pSubcode)
	PackFile = False
End Function

Private Sub showTable(byref fo, byref cnn, byval tableName)
	Dim rsTable
	Dim i
	Dim fieldCount
	Dim data
	Set rsTable = cnn.Execute("SELECT * FROM " & tableName & " ORDER BY 1")
	fieldCount = rsTable.Fields.count
	rsTable.MoveFirst
	Do While Not rsTable.EOF
		For i = 0 To fieldCount - 1
			if i = 0 Then
				data = rsTable(i).Value
			Else
				data = data & vbTab & rsTable(i).Value
			End If
		Next
		rsTable.MoveNext
		fo.WriteLine data
	Loop
	rsTable.Close
	Set rsTable = Nothing
End Sub

</script>
</scriptlet>

```  
  
３．WinMergeの展開プラグインを自動にするか、SqliteToText.sctを明示して比較  
を実行する。  
![無題.png](/image/8a540841-3442-3620-f8f6-19356f0faa10.png)  
  
このように、テーブル、トリガー、ビュー、インデックスの情報、とテーブルの中身を比較する。  
  
# 解説  
## SQLiteのメタ情報  
### sqlite_master  
SQLiteはsqlite_masterにtable,index,view,trrigerの情報を格納している。  
このテーブルを取得することで下記の情報が取得できる。  
  
|名前|説明|  
|:---|:---|  
|type|オブジェクトの情報を示す。'table', 'index', 'view','trigger'のいずれか|  
|name|オブジェクト名|  
|rootpage|テーブルとインデックスのためのroot b-treeページのページ番号|  
|sql|SQL|  
  
なお、このテーブルには一時テーブルの情報は格納されない。  
一時テーブルの情報を取得するには下記のテーブルからデータを取得する。  
  
・sqlite_temp_master  
  
  
### テーブルの列情報  
ここでは取得してないが、テーブルの列情報は下記のSQLで取得できる。  
  
```
PRAGMA table_info('テーブル名');
```  
  
### シーケンス情報  
AUTOINCREMENTをPRIMARYキーで作成した時にsqlite_sequenceが作成される。  
AUTOINCREMENTはsqlite_sequenceを元に作成される。  
  
### コマンドラインを利用したダンプ方法  
最新のコマンドラインツールがあるなら、以下の方法で似た情報が取得できる。  
  
```
sqlite3 test.sqlite .dump
```  
  
# 参考  
 __The SQLite Database File Format__   
http://www.sqlite.org/fileformat.html  
