# 前書き  
むかしむかし、ゆっくりさんにPowerPointを使って喋らせた動画を作成したことがあります。  
  
<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">Powerpointを使用したゆっくり動画の作成 <a href="https://t.co/nCQIQI6p7M">https://t.co/nCQIQI6p7M</a> <a href="https://twitter.com/hashtag/sm18775987?src=hash&amp;ref_src=twsrc%5Etfw">#sm18775987</a> <a href="https://twitter.com/hashtag/%E3%83%8B%E3%82%B3%E3%83%8B%E3%82%B3%E5%8B%95%E7%94%BB?src=hash&amp;ref_src=twsrc%5Etfw">#ニコニコ動画</a></p>&mdash; m.ita (@mima_ita) <a href="https://twitter.com/mima_ita/status/1157131068895424512?ref_src=twsrc%5Etfw">August 2, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>  
  
今回は、令和の時代になったので、ゆっくりさんには卒業してもらって、VOICEROIDE2の茜ちゃんにやっていただくこととします。  
  
# 仕組み  
![image.png](/image/a5a83de0-689d-3b1e-f320-dca091c9b4b1.png)  
PowerPointの各スライドのページに記載された文字をVOICEROIDE2を用いてWavファイルの作成を行います。その作成したWavファイルをスライドに埋め込みます。  
  
スライドショーを実行することで、スライドを切り替え後、マウスクリックをすることで、茜ちゃんがしゃべってくれます。  
  
  
# 使い方  
**環境:**  
Windows10  
Office16 PowerPoint(32bit)  
VOICEROIDE2  
  
(1)以下から「akanechan.pptm」ファイルをダウンロードするか、任意のpptmファイルのVBAに「src」フォルダのclsとbasファイルをインポートしてください。  
https://github.com/mima3/akanechan_powerpoint  
  
(2)スライドのノートに喋らせたい文字を入力します。  
![image.png](/image/ea401747-fb97-3e57-cc7c-d9134e9939ce.png)  
  
(3) VOICEROID2を起動します。  
  
(4)「AddSoftTalk」マクロを実行します。  
![image.png](/image/b805e5a0-2753-3f3f-d999-04740d48c326.png)  
  
これにより以下の処理が行われます。  
  
 - すべてのスライドのノートに書かれた文字が、改行毎にVOICEROID2におくられてWavファイルを作成します。このファイルはPowerPointと同じフォルダに存在します。  
 - 作成したWavファイルを各スライドに埋め込みます。  
  
(5)マクロが終了するとすべてのスライドに以下のような音声が埋め込まれていることが確認できます。  
![image.png](/image/2b92ac88-eaa9-6465-5552-fb0b1ef023c5.png)  
  
   
(6)スライドショーの記録を行います。この過程で埋め込んだ音声が再生されます。  
　記録されたスライドショーはファイルのエクスポートからビデオを作成することができます。  
  
# ソースコードの内容  
## VbaUiAuto.cls  
UIAutomationを操作する処理をまとめたものになります。  
  
**:VbaUiAuto.cls**  
```vb:VbaUiAuto.cls
Option Explicit
'* UIAutomationClientを参照設定してください。
Private uia As UIAutomationClient.CUIAutomation
Private Declare Sub Sleep Lib "kernel32" (ByVal dwMilliseconds As Long)

Private Sub Class_Initialize()
    Set uia = New UIAutomationClient.CUIAutomation
End Sub
Private Sub Class_Terminate()
    Set uia = Nothing
End Sub
Public Sub SleepMilli(ByVal millisec As Long)
    Call Sleep(millisec)
End Sub
' ルートのディスクトップ要素を取得
Public Function GetRoot() As IUIAutomationElement
    Dim ret As IUIAutomationElement
    Set ret = uia.GetRootElement
    Set GetRoot = ret
End Function

' 指定の子ウィンドウをタイトルから取得する
Public Function GetMainWindowByTitle(ByRef form As IUIAutomationElement, ByVal name As String) As IUIAutomationElement
    Dim cnd As IUIAutomationCondition
    Dim ret As IUIAutomationElement
    Set cnd = uia.CreatePropertyCondition(UIA_PropertyIds.UIA_NamePropertyId, name)

    Set ret = form.FindFirst(TreeScope_Element Or TreeScope_Children, cnd)

    Set GetMainWindowByTitle = ret
End Function

' 指定の子ウィンドウをタイトルから取得できまで待機
Public Function WaitMainWindowByTitle(ByRef form As IUIAutomationElement, ByVal name As String, ByVal timeOutSec As Double) As IUIAutomationElement
    Dim start As Variant
    start = Timer()
    Dim ret As IUIAutomationElement

    Set ret = GetMainWindowByTitle(form, name)
    Do While ret Is Nothing
        If Timer() - start > timeOutSec Then
            Exit Function
        End If
        Set ret = GetMainWindowByTitle(form, name)
        Call SleepMilli(100)
    Loop
    Set WaitMainWindowByTitle = ret
End Function

' ボタンを指定Indexを押下する
Public Sub pushButton(ByRef form As IUIAutomationElement, ByVal ix As Long)
    Dim cnd As IUIAutomationCondition
    Set cnd = uia.CreatePropertyCondition(UIA_PropertyIds.UIA_ClassNamePropertyId, "Button")

    Dim list As IUIAutomationElementArray
    Set list = form.FindAll(TreeScope_Element Or TreeScope_Descendants, cnd)

    Dim ptn As IUIAutomationInvokePattern
    Set ptn = list.GetElement(ix).GetCurrentPattern(UIA_PatternIds.UIA_InvokePatternId)
    Call ptn.Invoke

End Sub


' 指定のClassNameがTextBoxに値を設定する
Public Sub SetText(ByRef form As IUIAutomationElement, ByVal ix As Long, ByVal text As String)
    Dim cnd As IUIAutomationCondition
    Set cnd = uia.CreatePropertyCondition(UIA_PropertyIds.UIA_ClassNamePropertyId, "TextBox")

    Dim list As IUIAutomationElementArray
    Set list = form.FindAll(TreeScope_Element Or TreeScope_Descendants, cnd)


    Dim editValue As IUIAutomationValuePattern
    Set editValue = list.GetElement(ix).GetCurrentPattern(UIA_PatternIds.UIA_ValuePatternId)
    Call editValue.SetValue(text)

End Sub

' 指定のAutomationIdでTextBoxに値を設定する
Public Sub SetTextById(ByRef form As IUIAutomationElement, ByVal id As String, ByVal text As String)
    Dim cnd As IUIAutomationCondition
    Set cnd = uia.CreatePropertyCondition(UIA_PropertyIds.UIA_AutomationIdPropertyId, id)

    Dim list As IUIAutomationElementArray
    Set list = form.FindAll(TreeScope_Element Or TreeScope_Descendants, cnd)


    Dim editValue As IUIAutomationValuePattern
    Set editValue = list.GetElement(0).GetCurrentPattern(UIA_PatternIds.UIA_ValuePatternId)
    Call editValue.SetValue(text)

End Sub

' 指定のClassNameがTextBoxの値を取得する
Public Function GetStatusBarItemText(ByRef form As IUIAutomationElement, ByVal ix As Long) As String
    Dim cnd As IUIAutomationCondition
    Set cnd = uia.CreatePropertyCondition(UIA_PropertyIds.UIA_ClassNamePropertyId, "StatusBarItem")

    Dim list As IUIAutomationElementArray
    Set list = form.FindAll(TreeScope_Element Or TreeScope_Descendants, cnd)

    GetStatusBarItemText = list.GetElement(ix).CurrentName

End Function
```  
  
## VoiceRoid.cls  
VOICEROIDE2で音声ファイルを作成するための処理です。  
このあたりは[二番煎じ](https://qiita.com/mima_ita/items/d4655de865f30bb51c65#vba--uiautomation%E3%81%A7%EF%BD%B1%EF%BD%B6%EF%BE%88%EF%BE%81%EF%BD%AC%EF%BE%9D%EF%BD%B6%EF%BE%9C%EF%BD%B2%EF%BD%B2%EF%BE%94%EF%BD%AF%EF%BE%80%EF%BD%B0)になっています。  
  
  
**:VoiceRoid.cls**  
```vb:VoiceRoid.cls
Option Explicit
Private vua As VbaUiAuto
Private mainForm As IUIAutomationElement
'*
'* 初期化
'*
Private Sub Class_Initialize()
    Set vua = New VbaUiAuto
    Set mainForm = vua.GetMainWindowByTitle(vua.GetRoot(), "VOICEROID2")
    If (mainForm Is Nothing) Then
        Set mainForm = vua.GetMainWindowByTitle(vua.GetRoot(), "VOICEROID2*")
        If (mainForm Is Nothing) Then
            Err.Raise 999, "VoiceRoid.Init", "VOICEROIDE2が起動していない"
            Exit Sub
        End If
    End If
End Sub

'**
'* VOICEROID2によるWavファイルの作成
'* @param[in] msg しゃべる内容
'* @param[in] wavPath 作成するwavファイルのパス
'**
Public Sub CreateWavFile(ByVal msg As String, ByVal wavPath As String)
    ' 茜ちゃんのセリフ設定
    Call vua.SetText(mainForm, 0, msg)
    
    ' 音声保存
    Call vua.pushButton(mainForm, 4)

    ' 5秒以内に音声保存画面が表示されたら保存ボタンを押す
    Dim saveWvForm As IUIAutomationElement
    Set saveWvForm = vua.WaitMainWindowByTitle(mainForm, "音声保存", 5)
    Call vua.pushButton(saveWvForm, 0)

    ' 名前を付けて保存に日付のファイル名を作る
    Dim saveFileForm As IUIAutomationElement
    Set saveFileForm = vua.WaitMainWindowByTitle(saveWvForm, "名前を付けて保存", 5)
    Call vua.SetTextById(saveFileForm, "1001", wavPath)
    SendKeys "{ENTER}"

    ' 情報ポップアップのOKを押下
    Dim infoForm As IUIAutomationElement
    Set infoForm = vua.WaitMainWindowByTitle(saveWvForm, "情報", 60)
    Call vua.pushButton(infoForm, 0)

End Sub
```  
  
## Main.bas  
スライドショーのノート解析～VOICEROIDE2で作成したWavファイルの埋め込みを行っています。  
  
**:Main.bas**  
```vb:Main.bas
Option Explicit



'*
'* ノートに書いた内容をスライドの切り替え時にsofttalkを用いてしゃべるようにします。
'* 「画像切り替え」タブのサウンドにそのファイルは指定されています。
'* このリストに追加したファイルはPowerPointerを再起動したときに使用されていなければリストから消えます
'*
Public Sub AddSoftTalk()
    Dim sld As Slide
    Dim note As Slide
    Dim msg As String
    Dim wavPath As String
    ' Visit each slide
    For Each sld In ActivePresentation.Slides
        Call AddSoftTalkToSlide(sld)
    Next
End Sub
'*
'* 選択中のスライドに対して音声を追加する
'*
Public Sub AddSoftTalkToSelectedSlide()
    Dim sld As Slide
    Set sld = ActivePresentation.Slides.Item(ActiveWindow.Selection.SlideRange.SlideIndex)
    Call AddSoftTalkToSlide(sld)
End Sub

Private Sub AddSoftTalkToSlide(ByRef sld As Slide)
    Dim note As Slide
    Dim msg As String
    Dim wavPath As String
    Dim line As Variant
    
    Dim vr As VoiceRoid
    Set vr = New VoiceRoid
    
    Dim i As Long
    
    For Each note In sld.NotesPage
        msg = note.Shapes.Item(2).TextEffect.text
        Debug.Print msg
        If msg <> "" Then
            i = 0
            line = Split(msg, vbCr)
            For i = LBound(line) To UBound(line)
                If line(i) <> "" Then
                    wavPath = ActivePresentation.Path & "\" & sld.name & "_" & i & ".wav"
                    ' Wavファイル作成
                    Call vr.CreateWavFile(line(i), wavPath)
            
                    Call ApeendWavFile(sld, wavPath)
                End If
            Next i
        End If
    Next

End Sub


'**
'* スライドにファイルを追加します。
'* この際全てのShapesをチェックしてすでに追加されていないか確認します。
'* @param[in,out] sld 対象のスライド
'* @param[in] wavPath 作成するwavファイルのパス
'**
Private Sub ApeendWavFile(ByRef sld As Slide, ByVal wavPath As String)
    ' 重複チェック & 削除
    Dim shp As Shape
    Dim rmIndex As Long
    rmIndex = 0
    Dim i As Long
    i = 1
    For Each shp In sld.Shapes
        If shp.Type = msoMedia Then
            If shp.MediaType = ppMediaTypeSound Then
                If Dir(wavPath) = shp.name Then
                    rmIndex = i
                    Exit For
                End If
            End If
        End If
        i = i + 1
    Next
    If rmIndex <> 0 Then
        sld.Shapes.Item(rmIndex).Delete
    End If
    
    Set shp = sld.Shapes.AddMediaObject2(wavPath)
    shp.AnimationSettings.PlaySettings.PlayOnEntry = msoTrue
    shp.AnimationSettings.PlaySettings.HideWhileNotPlaying = msoTrue

    
End Sub

```  
  
# あとがき  
これで茜ちゃんとしてプレゼンテーションができるようになります。  
