# 目的  
MSProjectをVBScriptまたはVBAで操作する。  
  
なお、MSProjectの評価版は下記からダウンロードすることができる。  
http://technet.microsoft.com/ja-jp/evalcenter/default.aspx  
  
# リファレンス  
 __Project 2013オブジェクト モデルのリファレンス__   
http://msdn.microsoft.com/ja-jp/library/office/dn175492%28v=office.15%29.aspx  
  
 __Application and Project Object Map(Office 2010)__   
http://msdn.microsoft.com/en-us/library/office/ff863668%28v=office.14%29.aspx  
  
  
# VBAでの操作例  
MSProjectはExcelなどの他のオフィス製品と同様にVBAで操作を行うことができる。  
  
## タスクの選択  
プロジェクト中のタスクに参照する場合はProjectオブジェクトのTasksコレクションから参照する。  
Tasksコレクションは[Task](http://msdn.microsoft.com/ja-jp/library/office/office.vbapj.chm131329%28v=office.15%29.aspx "Task")のオブジェクトの集合だ。  
下記のようにタスク名またはID名を指定してタスクを取得できる。  
  
サンプル：  
  
```
    Dim prj As Project
    Set prj = ThisProject
    Debug.Print prj.tasks.item(7).id
    Debug.Print prj.tasks.item("TEST11").id
```  
  
## タスクのプロパティ  
タスクオブジェクトの詳細情報は下記のプロパティから設定、取得できる。  
  
|プロパティ名|説明|  
|:-----------|:---|  
|Name|タスクの名称。ProjectのTasksオブジェクトはこの名称をキーに管理されている。|  
|OutlineLevel|タスクの階層のレベル。親がいない場合は1になる。OutlineLevelを指定しても階層構造を変更することはできない。|  
|OutlineChildren|サブタスクの一覧。Taskオブジェクトのコレクションになっている。このコレクションに対してAddを行なっても階層構造を考慮した追加はされない。|  
|Notes|タスクに対する注釈を記述できる。|  
|Start|タスクの開始日|  
|Finish|タスクの終了日|  
|PercentComplete|達成率|  
|ActualStarte|開始日実績|  
|ActualFinish|終了日実績|  
|BaselineStart|タスクの割り当ての基準開始日|  
|BaselineFinish|タスクの割り当ての基準終了日|  
|ResourceNames|リソースの名称。複数存在する場合は,で区切る|  
|Resouces|Resourceオブジェクトのコレクション。|  
|PredecessorTasks|先行タスクのTaskオブジェクトのコレクション。存在しない場合は、先行タスクがないことをあらわす。|  
  
サンプル：  
  
```
    Dim prj As Project
    Set prj = ThisProject
    Dim tsk As task
    Dim pTsk As task
    Dim r As Resource
    For Each tsk In prj.tasks
        Debug.Print tsk.id
        Debug.Print "  Name:" & tsk.name
        Debug.Print "  OutlineLevel:" & tsk.OutlineLevel
        Debug.Print "  Notes:" & tsk.Notes
        Debug.Print "  Start:" & tsk.Start
        Debug.Print "  Finish:" & tsk.Finish
        Debug.Print "  PercentComplete:" & tsk.PercentComplete
        Debug.Print "  ActualStart:" & tsk.ActualStart
        Debug.Print "  ActualFinish:" & tsk.ActualFinish
        Debug.Print "  BaselineStart:" & tsk.BaselineStart
        Debug.Print "  BaselineFinish:" & tsk.BaselineFinish
        Debug.Print "  ResourceNames:" & tsk.ResourceNames
        Debug.Print "  Resources:"
        For Each r In tsk.Resources
            Debug.Print "    Resouce.Id :" & r.id
            Debug.Print "    Resouce.Name :" & r.name
        Next
        Debug.Print "  PredecessorTasks:"
        For Each pTsk In tsk.PredecessorTasks
            Debug.Print "    " & pTsk.id & "  " & pTsk.name
        Next
        ' Readonly
        Debug.Print "  OutlineChildren:"
        For Each pTsk In tsk.OutlineChildren
            Debug.Print "    " & pTsk.id & "  " & pTsk.name
        Next
    Next
```  
  
## タスクの追加  
  
末尾への追加例  
  
```
ThisProject.Tasks.Add "TaskName1"
```  
  
指定の位置に追加例  
  
```
ThisProject.Tasks.Add "TaskName2",3
```  
  
タスクの階層については自分で指定しなければならない。  
親となるタスクの直後に追加した後に、OutlineIndentまたは、OutlineOutdentを用いて階層を調整する必要がある。  
  
```
' 指定のdepth に階層をあわせる場合
            diffLv = depth - task.OutlineLevel
            For i = 1 To Abs(diffLv)
                If diffLv > 0 Then
                    Call task.OutlineIndent
                Else
                    Call task.OutlineOutdent
                End If
            Next i
```  
  
## タスクの削除  
TaskオブジェクトのDeleteメソッドを用いる  
全て削除する場合は下記のとおり  
  
```
    Dim tsk As task
    Dim i As Long
    For i = ThisProject.tasks.Count To 1 Step -1
        prjObj.tasks.item(i).Delete
    Next
```  
  
## リソースの割り当て  
TaskオブジェクトのResourceNamesプロパティまたはAssignmentsプロパティよりタスクにアサインしたリソースの追加を行なえる。  
  
```
task.ResourceNames = "owner"
```  
  
リソースID 212のリソースをタスクに割り当てる例  
  
```
ActiveProject.Tasks(1).Assignments.Add ResourceID:=212
```  
  
# VBScriptでの実行例  
"MSProject.Application"をCreateObjectすることで操作が可能になる。  
その他はVBAと同様の使用方法となる。  
  
以下の例は指定のプロジェクトの内容をテキストに書き込む例である。  
  
```
UnpackFile "C:\\Documents and Settings\\All Users\\Documents\\prjSample.mpp", "TEST.txt", True, True

Function UnpackFile(fileSrc, fileDst, pbChanged, pSubcode)
	Dim fo
	Dim fso
	Dim mp
	Dim prj
	Dim targetPrj
	Dim cmp
	Dim tsk
	Dim pTsk
	Dim r

	Set targetPrj = Nothing
	
	Set fso = CreateObject("Scripting.FileSystemObject")
	Set fo = fso.CreateTextFile(fileDst, True, True)

	Set mp = CreateObject("MSProject.Application")
	mp.DisplayAlerts = False
	mp.AutomationSecurity  = 3 ' msoAutomationSecurityForceDisable
	mp.FileOpenEx fileSrc, True

	For Each prj In mp.Projects
		If prj.Name = fso.GetFileName(fileSrc) Then
			Set targetPrj = prj
			Exit For
		End If
	Next
	If targetPrj Is Nothing Then
		Exit Function
	End If


	fo.WriteLine fileSrc
	fo.WriteLine fileDst
	fo.WriteLine targetPrj.Name
	For Each tsk In targetPrj.tasks
		fo.WriteLine "TaskID:" & tsk.id
		fo.WriteLine "  Name:" & tsk.name
		fo.WriteLine "  OutlineLevel:" & tsk.OutlineLevel
		fo.WriteLine "  Notes:" & tsk.Notes
		fo.WriteLine "  Start:" & tsk.Start
		fo.WriteLine "  Finish:" & tsk.Finish
		fo.WriteLine "  PercentComplete:" & tsk.PercentComplete
		fo.WriteLine "  ActualStart:" & tsk.ActualStart
		fo.WriteLine "  ActualFinish:" & tsk.ActualFinish
		fo.WriteLine "  BaselineStart:" & tsk.BaselineStart
		fo.WriteLine "  BaselineFinish:" & tsk.BaselineFinish
		fo.WriteLine "  ResourceNames:" & tsk.ResourceNames
		fo.WriteLine "  Resources:"
		For Each r In tsk.Resources
			fo.WriteLine "    Resouce.Id :" & r.id
			fo.WriteLine "    Resouce.Name :" & r.name
		Next
		fo.WriteLine "  PredecessorTasks:"
		For Each pTsk In tsk.PredecessorTasks
			fo.WriteLine "    " & pTsk.id & "  " & pTsk.name
		Next
		' Readonly
		fo.WriteLine "  OutlineChildren:"
		For Each pTsk In tsk.OutlineChildren
			fo.WriteLine "    " & pTsk.id & "  " & pTsk.name
		Next
	Next

	For Each cmp In targetPrj.VBProject.VBComponents
		fo.WriteLine "[CodeModule." & cmp.Name & "]"
		If cmp.CodeModule.CountOfLines > 0 Then
			fo.WriteLine cmp.CodeModule.Lines(1, cmp.CodeModule.CountOfLines)
		End If
		fo.WriteLine ""
	Next
	mp.Quit
	Set mp = Nothing
	Set prj = Nothing

	fo.Close
	Set fo = Nothing
	Set fso = Nothing

	pbChanged = True
	pSubcode = 0
	UnpackFile = True

End Function
```  
  
# 応用例  
 __TracのチケットからMsProjectのタスクをインポートする__   
http://needtec.exblog.jp/21472581/  
  
 __WinMergeのMSProject用のプラグイン__   
https://github.com/mima3/MppToText  
