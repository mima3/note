# 目的  
この記事は、**IT企業**を名乗る名状しがたい企業においてRedmineとかTracを使用した、タスク管理の導入を、あきらめた方が対象です。  
  
今回は、彼らのルールにしたがったOfficeという土俵で多少マシな状況を作るためにPowerShellを用いてOutlookの自動操作を行う方法を調べてみました。  
  
Outlookはメール送るだけでなく、タスクの依頼や会議の設定ができます。  
これらの操作はVBAやVBS、そしてPowerShellによって自動化が可能になっています。  
  
すくなくとも一つのExcelを全員で修正したり、タスクの変更は気を付けて確認するという人間の能力を過大評価して現在のIT技術を過少評価した、タスク管理っぽいなにかをしているとこでは、多少はマシになる可能性があると期待しています。  
  
**環境：**  
　Office16　Outlook(32bit)  
　Windows 10  
　PowerShell 5.1  
  
  
Outlookオブジェクトの解放処理について冗長な書き方をしていますが、その背景は下記を参照してください。  
  
 - [.NETを使ったOfficeの自動化が面倒なはずがない―そう考えていた時期が俺にもありました。](https://qiita.com/mima_ita/items/aa811423d8c4410eca71)  
  
  
# メールの操作  
## メールの送信サンプル  
指定の画像ファイルを添付ファイルとしてメールを送信する例になります。  
  
```powershell
	# https://docs.microsoft.com/ja-jp/office/vba/api/overview/outlook
	# https://community.spiceworks.com/how_to/150253-send-mail-from-powershell-using-outlook
	$app = New-Object -ComObject Outlook.Application

	# プロファイルを選択してログオン
	$namespace = $app.GetNamespace("MAPI")
	$namespace.Logon("Outlook")

	# https://docs.microsoft.com/ja-jp/office/vba/api/outlook.olitemtype
	# 0:olmailitem
	$mail = $app.CreateItem(0)
	$mail.To = "送信先メールアドレス" 
	$mail.Subject = "タイトルでごわす" 
	$mail.Body = "本文でごわす"
	$attachments = $mail.Attachments
	$addResult = $attachments.Add("C:\dev\ps\mail\test.png")
	$mail.Send()

	# 送受信を行う
	# このメソッドは非同期なので、10秒ほどスリープしている
	# https://docs.microsoft.com/ja-jp/office/vba/api/outlook.namespace.sendandreceive
	$session = $app.Session
	$session.SendAndReceive($False)
	Start-Sleep 10

	#
	$namespace.Logoff()

	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($namespace) | Out-Null
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($addResult) | Out-Null
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($attachments) | Out-Null
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($Mail) | Out-Null
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($namespace) | Out-Null
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($session) | Out-Null


	$namespace = $null
	Remove-Variable namespace -ErrorAction SilentlyContinue
	$addResult = $null
	Remove-Variable addResult -ErrorAction SilentlyContinue
	$attachments = $null
	Remove-Variable attachments -ErrorAction SilentlyContinue
	$mail = $null
	Remove-Variable mail -ErrorAction SilentlyContinue
	$namespace = $null
	Remove-Variable namespace -ErrorAction SilentlyContinue
	$session = $null
	Remove-Variable session -ErrorAction SilentlyContinue

	[System.GC]::Collect()
	[System.GC]::WaitForPendingFinalizers()
	[System.GC]::Collect()

	$app.Quit() 


	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($app) | Out-Null
	$app = $null
	Remove-Variable app -ErrorAction SilentlyContinue

	[System.GC]::Collect()
	[System.GC]::WaitForPendingFinalizers()
	[System.GC]::Collect()


```  
  
![image.png](/image/90d35564-6412-7ec2-8d40-0b374d32a190.png)  
  
## メールの受信フォルダ中のアイテムの列挙  
受信フォルダ内のメールを列挙します。  
サブフォルダの中身も列挙します。  
  
```powershell

	function DumpMailItem($folder, $prefix) {
	    Write-Host $prefix "------------------"
	    Write-Host $prefix $folder.Name
	    $prefix = "  " + $prefix
	    $items = $folder.Items

	    # メールアイテムの列挙
	    for ($i=1; $i -le $items.Count; $i++){
	       $item = $items.Item($i)
	       $sender = $item.Sender
	       if ($sender -eq $null) 
	       {
	           Write-Host $prefix  $item.SentOn    $item.Subject
	       } else {
	           Write-Host $prefix  $item.SentOn    $item.SenderName $sender.Address   $item.Subject
	           [System.Runtime.Interopservices.Marshal]::ReleaseComObject($sender) | Out-Null
	       }
	       [System.Runtime.Interopservices.Marshal]::ReleaseComObject($item) | Out-Null
	    }

	    # フォルダの列挙
	    $folders = $folder.Folders
	    for ($i=1; $i -le $folders.Count; $i++){
	        $item = $folders.Item($i)
	        DumpMailItem $item $prefix
	        [System.Runtime.Interopservices.Marshal]::ReleaseComObject($item) | Out-Null
	    }
	    [System.Runtime.Interopservices.Marshal]::ReleaseComObject($items) | Out-Null
	    [System.Runtime.Interopservices.Marshal]::ReleaseComObject($folders) | Out-Null
	    [System.Runtime.Interopservices.Marshal]::ReleaseComObject($folder) | Out-Null
	}


	# https://docs.microsoft.com/ja-jp/office/vba/api/overview/outlook
	# https://community.spiceworks.com/how_to/150253-send-mail-from-powershell-using-outlook
	$app = New-Object -ComObject Outlook.Application

	# プロファイルを選択してログオン
	$namespace = $app.GetNamespace("MAPI")
	$namespace.Logon("Outlook")

	# このメソッドは非同期
	# https://docs.microsoft.com/ja-jp/office/vba/api/outlook.namespace.sendandreceive
	$session = $app.Session
	$session.SendAndReceive($True)
	Start-Sleep 10

	#https://docs.microsoft.com/ja-jp/office/vba/api/outlook.oldefaultfolders
	# 6: The Inbox folder.
	$inbox = $namespace.GetDefaultFolder(6)

	DumpMailItem $inbox "  "


	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($inbox) | Out-Null
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($session) | Out-Null
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($namespace) | Out-Null

	$inbox = $null
	Remove-Variable inbox -ErrorAction SilentlyContinue
	$session = $null
	Remove-Variable session -ErrorAction SilentlyContinue
	$namespace = $null
	Remove-Variable namespace -ErrorAction SilentlyContinue

	#
	[System.GC]::Collect()
	[System.GC]::WaitForPendingFinalizers()
	[System.GC]::Collect()

	#
	$app.Quit()

	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($app) | Out-Null
	$app = $null
	Remove-Variable app -ErrorAction SilentlyContinue

	[System.GC]::Collect()
	[System.GC]::WaitForPendingFinalizers()
	[System.GC]::Collect()
```  
  
フォルダの構成：  
![image.png](/image/1d192e01-f2be-c1b3-2aaa-72938703b61d.png)  
  
出力例：  
  
```
   ------------------
   受信トレイ
      Task Declined: タスクだお
      Task Declined: タスクだお
      Task Declined: タスクだお
     2019/08/22 1:24:20 Google Play noreply-developer-googleplay@google.com Google Play Developer Program Policy Update
     ------------------
     フォルダ
       2019/08/20 8:52:28 Facebook noreply@developers.facebook.com 開発者ブログウィークリーダイジェスト
       ------------------
       サブ１
         2019/08/20 14:47:06 test@co.jp test@co.jp subject
```  
  
# 予定の操作  
## 予定の追加  
  
```powershell
	# https://docs.microsoft.com/ja-jp/office/vba/api/overview/outlook
	# https://community.spiceworks.com/how_to/150253-send-mail-from-powershell-using-outlook
	$app = New-Object -ComObject Outlook.Application

	# プロファイルを選択してログオン
	$namespace = $app.GetNamespace("MAPI")
	$namespace.Logon("Outlook")


	# https://docs.microsoft.com/ja-jp/office/vba/api/outlook.olitemtype
	# olappointmentItem	1	AppointmentItem オブジェクト。

	# https://docs.microsoft.com/ja-jp/office/vba/api/outlook.appointmentitem
	$appointment = $app.CreateItem(1)
	$appointment.Subject = "タイトルだお" 
	$appointment.Body = "私は猫"
	$appointment.Location = "家"
	$appointment.ReminderSet = $True
	$appointment.ReminderMinutesBeforeStart = 30
	$appointment.Start = Get-Date -Date '2019/8/20 18:00:00'
	$appointment.End = Get-Date -Date '2019/8/20 18:30:00'

	$appointment.Save()

	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($appointment) | Out-Null
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($namespace) | Out-Null
	$appointment = $null
	Remove-Variable appointment -ErrorAction SilentlyContinue
	$namespace = $null
	Remove-Variable namespace -ErrorAction SilentlyContinue

	[System.GC]::Collect()
	[System.GC]::WaitForPendingFinalizers()
	[System.GC]::Collect()

	# 起動時に全て開いている場合、ここで待機されてしまう。(数十秒後にメールを送信して終了される）
	$app.Quit() 


	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($app) | Out-Null
	$app = $null
	Remove-Variable app -ErrorAction SilentlyContinue

	[System.GC]::Collect()
	[System.GC]::WaitForPendingFinalizers()
	[System.GC]::Collect()

```  
  
![image.png](/image/fe7ffc48-d9d9-2710-a97d-f018e673435b.png)  
![image.png](/image/7a484a43-00f4-3b2a-b824-76e29971e3c9.png)  
  
## 会議の予約  
  
```powershell
	# https://docs.microsoft.com/ja-jp/office/vba/api/overview/outlook
	# https://community.spiceworks.com/how_to/150253-send-mail-from-powershell-using-outlook
	$app = New-Object -ComObject Outlook.Application

	# プロファイルを選択してログオン
	$namespace = $app.GetNamespace("MAPI")
	$namespace.Logon("Outlook")

	# https://docs.microsoft.com/ja-jp/office/vba/api/outlook.olitemtype
	# olAppointmentItem	1	An AppointmentItem object.
	# https://docs.microsoft.com/ja-jp/office/vba/api/outlook.meetingitem
	$meeting = $app.CreateItem(1)

	#https://docs.microsoft.com/ja-jp/office/vba/api/outlook.appointmentitem.meetingstatus
	#olMeeting	1	The meeting has been scheduled.
	#olMeetingCanceled	5	The scheduled meeting has been cancelled.
	#olMeetingReceived	3	The meeting request has been received.
	#olMeetingReceivedAndCanceled	7	The scheduled meeting has been cancelled but still appears on the user's calendar.
	#olNonMeeting	0	An Appointment item without attendees has been scheduled. This status can be used to set up holidays on a calendar.
	$meeting.MeetingStatus = 1


	$meeting.Subject = "牛丼友会会合" 
	$meeting.Location = "吉野家"
	$meeting.Body = "吉野家"
	$meeting.Start = Get-Date -Date '2019/8/31 18:00:00'
	$meeting.Duration = 90

	$recipients = $meeting.Recipients
	$required = $recipients.Add("メールアドレス１")
	#https://docs.microsoft.com/ja-jp/office/vba/api/outlook.olmeetingrecipienttype
	# olOptional:2 Optional attendee
	$required.Type = 2

	$optional = $recipients.Add("メールアドレス２")
	# olRequired:1 Required attendee
	$optional.Type = 1

	$recipients.ResolveAll()
	$meeting.Send()

	#
	$session = $app.Session
	$session.SendAndReceive($False)
	Start-Sleep 10

	$namespace.Logoff()


	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($optional) | Out-Null
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($required) | Out-Null
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($recipients) | Out-Null
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($meeting) | Out-Null
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($namespace) | Out-Null
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($session) | Out-Null


	$optional = $null
	Remove-Variable optional -ErrorAction SilentlyContinue

	$required = $null
	Remove-Variable required -ErrorAction SilentlyContinue

	$recipients = $null
	Remove-Variable recipients -ErrorAction SilentlyContinue

	$meeting = $null
	Remove-Variable meeting -ErrorAction SilentlyContinue

	$namespace = $null
	Remove-Variable namespace -ErrorAction SilentlyContinue

	$session = $null
	Remove-Variable session -ErrorAction SilentlyContinue
	#
	[System.GC]::Collect()
	[System.GC]::WaitForPendingFinalizers()
	[System.GC]::Collect()

	$app.Quit() 
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($app) | Out-Null
	$app = $null
	Remove-Variable app -ErrorAction SilentlyContinue
	#
	[System.GC]::Collect()
	[System.GC]::WaitForPendingFinalizers()
	[System.GC]::Collect()
```  
  
出席依頼された人のスケジュール：  
  
![image.png](/image/725fb818-a1e9-ef2a-4415-87ed928eb16e.png)  
  
![image.png](/image/31854032-2b79-3aa4-3e1f-57d5c08fdfd3.png)  
  
## 予定の取得  
  
```powershell


	# https://docs.microsoft.com/ja-jp/office/vba/api/overview/outlook
	# https://community.spiceworks.com/how_to/150253-send-mail-from-powershell-using-outlook
	$app = New-Object -ComObject Outlook.Application

	# プロファイルを選択してログオン
	$namespace = $app.GetNamespace("MAPI")
	$namespace.Logon("Outlook")


	#https://docs.microsoft.com/ja-jp/office/vba/api/outlook.oldefaultfolders
	# 9: olFolderCalendar The Calendar folder.
	$calendar = $namespace.GetDefaultFolder(9)
	$items = $calendar.Items
	$items.Sort("[Start]")
	$items.IncludeRecurrences = $True
	$today = Get-Date -Format "yyyy/MM/dd"
	$filter = "[Start] >= '" + $today + "'"
	$item = $items.Find($filter)
	while ($item -ne $null) {
		Write-Host $item.Subject $item.Location $item.Start $item.End
		[System.Runtime.Interopservices.Marshal]::ReleaseComObject($item) | Out-Null
		$item = $items.FindNext()
	}


	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($items) | Out-Null
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($calendar) | Out-Null
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($namespace) | Out-Null

	$items = $null
	Remove-Variable items -ErrorAction SilentlyContinue
	$calendar = $null
	Remove-Variable calendar -ErrorAction SilentlyContinue
	$namespace = $null
	Remove-Variable namespace -ErrorAction SilentlyContinue

	#
	[System.GC]::Collect()
	[System.GC]::WaitForPendingFinalizers()
	[System.GC]::Collect()

	#
	$app.Quit()

	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($app) | Out-Null
	$app = $null
	Remove-Variable app -ErrorAction SilentlyContinue

	[System.GC]::Collect()
	[System.GC]::WaitForPendingFinalizers()
	[System.GC]::Collect()

```  
  
出力例：  
  
```
定期予定  2019/08/28 8:00:00 2019/08/28 8:30:00
定期予定  2019/08/29 8:00:00 2019/08/29 8:30:00
タイトルだお 家 2019/08/29 18:00:00 2019/08/29 18:30:00
定期予定  2019/09/04 8:00:00 2019/09/04 8:30:00
定期予定  2019/09/05 8:00:00 2019/09/05 8:30:00
定期予定  2019/09/11 8:00:00 2019/09/11 8:30:00
```  
  
# タスクの操作  
## 自分のタスクを追加する場合  
  
```powershell
	# https://docs.microsoft.com/ja-jp/office/vba/api/overview/outlook
	# https://community.spiceworks.com/how_to/150253-send-mail-from-powershell-using-outlook
	$app = New-Object -ComObject Outlook.Application

	# プロファイルを選択してログオン
	$namespace = $app.GetNamespace("MAPI")
	$namespace.Logon("Outlook")

	# https://docs.microsoft.com/ja-jp/office/vba/api/outlook.olitemtype
	# olTaskItem	3	A TaskItem object.
	# https://docs.microsoft.com/ja-jp/office/vba/api/outlook.taskitem
	$task = $app.CreateItem(3)
	$task.Subject = "タスクだお" 
	$task.Body = "私は猫"
	$task.StartDate = Get-Date -Date '2019/8/30 18:00:00'
	$task.DueDate  = Get-Date -Date '2019/8/31 18:30:00'

	# 優先度　0:低 1:中 2:高
	$task.Importance = 0
	$task.Save()

	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($namespace) | Out-Null
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($task) | Out-Null

	$namespace = $null
	Remove-Variable namespace -ErrorAction SilentlyContinue

	$task = $null
	Remove-Variable task -ErrorAction SilentlyContinue


	[System.GC]::Collect()
	[System.GC]::WaitForPendingFinalizers()
	[System.GC]::Collect()

	$app.Quit() 

	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($app) | Out-Null
	$app = $null
	Remove-Variable app -ErrorAction SilentlyContinue

	[System.GC]::Collect()
	[System.GC]::WaitForPendingFinalizers()
	[System.GC]::Collect()

```  
![image.png](/image/dbc22563-f143-bd03-343d-08745efad5cf.png)  
  
![image.png](/image/9ba4f42a-cc61-f93b-11cd-829009ad1920.png)  
  
## タスクを依頼する場合  
  
```powershell
	# https://docs.microsoft.com/ja-jp/office/vba/api/overview/outlook
	# https://community.spiceworks.com/how_to/150253-send-mail-from-powershell-using-outlook
	$app = New-Object -ComObject Outlook.Application

	# プロファイルを選択してログオン
	$namespace = $app.GetNamespace("MAPI")
	$namespace.Logon("Outlook")

	# https://docs.microsoft.com/ja-jp/office/vba/api/outlook.olitemtype
	# olTaskItem	3	A TaskItem object.
	# https://docs.microsoft.com/ja-jp/office/vba/api/outlook.taskitem
	$task = $app.CreateItem(3)

	$task.Subject = "タスクの依頼" 
	$task.Body = "私はカモメ"
	$task.StartDate = Get-Date -Date '2019/8/30 18:00:00'
	$task.DueDate  = Get-Date -Date '2019/8/31 18:30:00'

	# 優先度　0:低 1:中 2:高
	$task.Importance = 0

	$assignResult = $task.Assign()
	$recipients = $task.Recipients
	$addResult = $recipients.Add("送信先メール")

	$recipients.ResolveAll()

	$task.Send()
	#$task.Save()
	# https://docs.microsoft.com/ja-jp/office/vba/api/outlook.olinspectorclose
	# 

	#
	$session = $app.Session
	$session.SendAndReceive($False)
	$namespace.Logoff()

	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($recipients) | Out-Null
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($addResult) | Out-Null
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($assignResult) | Out-Null
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($task) | Out-Null
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($namespace) | Out-Null
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($session) | Out-Null

	$recipients = $null
	Remove-Variable recipients -ErrorAction SilentlyContinue

	$addResult = $null
	Remove-Variable addResult -ErrorAction SilentlyContinue

	$assignResult = $null
	Remove-Variable assignResult -ErrorAction SilentlyContinue

	$task = $null
	Remove-Variable task -ErrorAction SilentlyContinue

	$namespace = $null
	Remove-Variable namespace -ErrorAction SilentlyContinue

	$session = $null
	Remove-Variable session -ErrorAction SilentlyContinue

	[System.GC]::Collect()
	[System.GC]::WaitForPendingFinalizers()
	[System.GC]::Collect()

	$app.Quit() 

	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($app) | Out-Null

	$app = $null
	Remove-Variable app -ErrorAction SilentlyContinue
		
	[System.GC]::Collect()
	[System.GC]::WaitForPendingFinalizers()
	[System.GC]::Collect()

```  
  
依頼先のタスク  
![image.png](/image/0a4f6e59-f1c4-4a98-b8e6-d98a0ff4f238.png)  
  
承認を押下することで進捗率等が変更できるようになる。  
![image.png](/image/b52aa462-0269-9688-03e9-cd3cb51f6544.png)  
  
進捗レポートの送信をすることで、依頼者のタスクが更新される。  
![image.png](/image/0dbdccae-368b-09f5-807e-ad2f5703e8e5.png)  
  
## タスクの列挙  
  
```powershell
	function DumpMailItem($folder, $prefix) {
	    Write-Host $prefix "------------------"
	    Write-Host $prefix $folder.Name
	    $prefix = "  " + $prefix
	    $items = $folder.Items

	    # タスクの列挙
	    for ($i=1; $i -le $items.Count; $i++){
	        $item = $items.Item($i)
	        #OlTaskStatus:https://docs.microsoft.com/ja-jp/office/vba/api/outlook.oltaskstatus
			#Name	Value	Description
			#olTaskComplete	2	The task is complete.
			#olTaskDeferred	4	The task is deferred.
			#olTaskInProgress	1	The task is in progress.
			#olTaskNotStarted	0	The task has not yet started.
			#olTaskWaiting	3	The task is waiting on someone else.	
	        Write-Host $prefix  $item.Subject $item.Owner $item.DueDate $item.Complete $item.Status $item.PercentComplete
	        [System.Runtime.Interopservices.Marshal]::ReleaseComObject($item) | Out-Null
	    }

	    # フォルダの列挙
	    $folders = $folder.Folders
	    for ($i=1; $i -le $folders.Count; $i++){
	        $item = $folders.Item($i)
	        DumpMailItem $item $prefix
	        [System.Runtime.Interopservices.Marshal]::ReleaseComObject($item) | Out-Null
	    }
	    [System.Runtime.Interopservices.Marshal]::ReleaseComObject($items) | Out-Null
	    [System.Runtime.Interopservices.Marshal]::ReleaseComObject($folders) | Out-Null
	    [System.Runtime.Interopservices.Marshal]::ReleaseComObject($folder) | Out-Null
	}


	# https://docs.microsoft.com/ja-jp/office/vba/api/overview/outlook
	# https://community.spiceworks.com/how_to/150253-send-mail-from-powershell-using-outlook
	$app = New-Object -ComObject Outlook.Application

	# プロファイルを選択してログオン
	$namespace = $app.GetNamespace("MAPI")
	$namespace.Logon("Outlook")

	# このメソッドは非同期
	# https://docs.microsoft.com/ja-jp/office/vba/api/outlook.namespace.sendandreceive
	$session = $app.Session
	$session.SendAndReceive($False)
	Start-Sleep 10

	#https://docs.microsoft.com/ja-jp/office/vba/api/outlook.oldefaultfolders
	# olFolderTasks	13	The Tasks folder.
	$taskfolder = $namespace.GetDefaultFolder(13)

	DumpMailItem $taskfolder "  "


	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($taskfolder) | Out-Null
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($session) | Out-Null
	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($namespace) | Out-Null

	$taskfolder = $null
	Remove-Variable taskfolder -ErrorAction SilentlyContinue
	$session = $null
	Remove-Variable session -ErrorAction SilentlyContinue
	$namespace = $null
	Remove-Variable namespace -ErrorAction SilentlyContinue

	#
	[System.GC]::Collect()
	[System.GC]::WaitForPendingFinalizers()
	[System.GC]::Collect()

	#
	$app.Quit()

	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($app) | Out-Null
	$app = $null
	Remove-Variable app -ErrorAction SilentlyContinue

	[System.GC]::Collect()
	[System.GC]::WaitForPendingFinalizers()
	[System.GC]::Collect()

```  
  
出力例  
  
```
   ------------------
   タスク
     タスクの依頼 ZZZZZ@YYYYY.ne.jp 2019/08/31 0:00:00 False 1 50
     ------------------
     サブタスク
       タスクだお XXXX@YYYYY.ne.jp 2019/08/31 0:00:00 False 0 0
       ------------------
       さらにサブタスク
         owatta ｘｘｘ@YYYYY.ne.jp 4501/01/01 0:00:00 True 2 100
```  
  
  
# まとめ  
Outlookをつかってメール送信やタスク管理の自動化が行えることはわかりました。  
おそらく、Excelでタスク管理をしているとこでも変更したタスクのメールの自動送信から初めて、Outlookのタスク機能を使う方向に徐々に倒していけば、マシになるかもしれません。  
