## 目的  
VBScriptでエンコードを指定してファイルを連結するスクリプトを記述する  
  
## スクリプト  
  
**:jointext.vbs**  
```vbnet:jointext.vbs

'* 指定ディレクトリの検索を行う
'* @param [in] searchPath 検索対象のパス
'* @param [out] fileList  パスを格納するコレクション
'* @param [in] targetExt  検索対象の拡張子
'* @param [in] excludeDir 検査対象のディレクトリ名
Sub SearchFile(ByVal searchPath , ByRef fileList, ByRef targetExt, ByRef excludeDir)
    Dim fso
    Dim oFile
    Dim oDir
    Dim oSubDir
    Set fso = CreateObject("Scripting.FileSystemObject")
    Set oDir = fso.GetFolder(searchPath)

    For Each oSubDir In oDir.SubFolders
        If Not excludeDir.Contains(oSubDir.Name) Then
            SearchFile searchPath & oSubDir.Name & "\", fileList, targetExt, excludeDir
        End If
    Next

    For Each oFile In oDir.Files
        If targetExt.Contains(fso.GetExtensionName(oFile.Name)) Then
            fileList.add searchPath & oFile.Name
        End If
    Next

End Sub

'* エンコードを指定してファイルを読んで出力ストリームに書き込む
'* @param[in] path 入力パス
'* @param[in] enc  エンコード
'* @param[out] outStream 出力ストリーム
Function ReadAndAppend(ByVal path, Byval enc, Byref outStream)
    Dim inStream
    Set inStream = CreateObject("ADODB.Stream")
    inStream.type = 2            '1:バイナリデータ 2:テキストデータ
    inStream.charset = enc       '入力ファイルの文字コード設定
    inStream.open
    inStream.LoadFromFile path   '入力ファイルを読み込む

    outStream.WriteText "// copy... " & path, 1
    outStream.WriteText inStream.ReadText(-1), 1   'WriteTextの第二引数：0:文字列のみ書き込む 1:文字列＋改行を書き込む

    inStream.Close
    Set inStream = Nothing

End Function

Dim objParm
Dim searchPath
Dim fileList
Dim excludeDir
Dim targetExt
Dim enc
Dim outputFileName
Set fileList = CreateObject("System.Collections.ArrayList")
Set excludeDir = CreateObject("System.Collections.ArrayList")
Set targetExt = CreateObject("System.Collections.ArrayList")
Set objParm = Wscript.Arguments
If objParm.Count < 3 Then
   WScript.echo "CScript jointext 検査対象のフォルダ 出力パス エンコード"
   WScript.echo "例： CScript jointext.vbs C:\dev\WSH\jointext\js\ out2.txt UTF-8"
   WScript.Quit
End If

targetExt.add "js"

excludeDir.add ".svn"
excludeDir.add ".git"

searchPath = objParm(0)
outputFileName = objParm(1)
enc = objParm(2)

SearchFile searchPath, fileList, targetExt, excludeDir

fileList.Sort

Dim outStream
Set outStream =CreateObject("ADODB.Stream")
outStream.type = 2
outStream.charset = enc '出力ファイルの文字コード設定
outStream.open 


Dim f
For Each f In fileList
   WScript.Echo f
   ReadAndAppend f, enc, outStream
Next

outStream.SaveToFile outputFileName, 2
outStream.Close
Set outStream = Nothing

```  
  
## 実行例  
  
```
CScript jointext.vbs C:\dev\WSH\jointext\js\ out2.txt UTF-8"
```  
  
C:\dev\WSH\jointext\js\ 以下のJavaScriptを連結して出力する。  
この時、.gitや.svnはフォルダは見ない。  
  
  
