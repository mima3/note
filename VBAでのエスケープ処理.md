通信を行なう際に、JavaScriptのescape関数は英数字以外の記号や日本語を％21とか％u3044というコードに変換してくれる。  
  
以下のページでは、VBAでesacape処理を行うサンプルを示している。  
http://www.xtremevbtalk.com/showthread.php?t=152882  
  
残念なことに、このサンプルは日本語などのUNICODEについては考慮されていない。そこで、上記のサンプルを元に日本語対応したものを以下に示す。  
  
```vbnet
Option Explicit

'*
'* 特殊文字をエスケープする
'* 参考：
'* http://www.xtremevbtalk.com/showthread.php?t=152882
'* @param[in] StringToEncode 文字
'* @return エスケープした文字 Unicodeの場合は%u
'*
Public Function escape(ByVal StringToEncode As String) As String
    Dim i As Integer
    Dim acode As Integer
    Dim char As String
 
    escape = StringToEncode
 
    For i = Len(escape) To 1 Step -1
        acode = AscW(Mid$(escape, i, 1))
        Select Case acode
            Case 48 To 57, 65 To 90, 97 To 122
                ' don't touch alphanumeric chars
 
            Case 32
                ' replace space
                escape = Left$(escape, i - 1) & "%20" & Mid$(escape, i + 1)
 
            Case Else
                ' replace punctuation chars with "%hex"
                char = Hex$(acode)
                If Len(char) > 2 Then
                    If Len(char) = 3 Then
                        char = "0" & char
                    End If
                    escape = Left$(escape, i - 1) & "%u" & char & Mid$(escape, i + 1)
                Else
                    If Len(char) = 1 Then
                        char = "0" & char
                    End If
                    escape = Left$(escape, i - 1) & "%" & char & Mid$(escape, i + 1)
                End If
        End Select
    Next
End Function
'* http://www.xtremevbtalk.com/showthread.php?t=152882
Public Function unescape(ByVal StringToDecode As String) As String
    Dim i As Long
    Dim acode As Integer, sTmp As String
    unescape = StringToDecode
 
    If InStr(1, unescape, "%") = 0 Then
         Exit Function
    End If
    For i = Len(unescape) To 1 Step -1
        acode = Asc(Mid$(unescape, i, 1))
        Select Case acode
            Case 48 To 57, 65 To 90, 97 To 122
                ' don't touch alphanumeric chars
                DoEvents
 
            Case 37: ' Decode % value
                If Mid$(unescape, i + 1, 1) = "u" Then
                    sTmp = CStr(CLng("&H" & Mid$(unescape, i + 2, 4)))
                    If IsNumeric(sTmp) Then
                        unescape = Left$(unescape, i - 1) & ChrW$(CInt("&H" & Mid$(unescape, i + 2, 4))) & Mid$(unescape, i + 6)
                    End If
                Else
                    sTmp = CStr(CLng("&H" & Mid$(unescape, i + 1, 2)))
                    If IsNumeric(sTmp) Then
                        unescape = Left$(unescape, i - 1) & ChrW$(CInt("&H" & Mid$(unescape, i + 1, 2))) & Mid$(unescape, i + 3)
                    End If
                End If
        End Select
    Next
End Function

```  
  
  
実行例:  
  
```
?escape("あたしはかこめabcdefg%#?<>*+[]<>/\'""")
%u3042%u305F%u3057%u306F%u304B%u3053%u3081abcdefg%25%23%3F%3C%3E%2A%2B%5B%5D%3C%3E%2F%5C%27%22

?unescape("%u3042%u305F%u3057%u306F%u304B%u3053%u3081abcdefg%25%23%3F%3C%3E%2A%2B%5B%5D%3C%3E%2F%5C%27%22")
あたしはかこめabcdefg%#?<>*+[]<>/\'"
```  
