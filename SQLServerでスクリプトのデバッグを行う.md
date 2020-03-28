このドキュメントではSQLServer 2008以降でストアドプロシージャのデバッグを行う方法について説明する。  
  
# 手順  
１. Microsoft SQL Server Management Studioでストアドプロシージャを右クリックする。  
２. [ストアドプロシージャをスクリプト化]⇒[EXECUTE]⇒新しいクエリーエディターウィンドウ  
３ 以下のようなスクリプトが生成される  
  
```sql
USE [Sample001]
GO

DECLARE @RC int
DECLARE @from int
DECLARE @to int

-- 方法: パラメーター値をここに指定します。

EXECUTE @RC = [dbo].[test_sp] 
   @from
  ,@to
GO

```  
  
４. テスト用のパラメータを入力する  
  
```sql
-- 方法: パラメーター値をここに指定します。
SET @from = 10
SET @to = 200
```  
  
５. スクリプトにブレークポイントを張る  
![sql.png](/image/832e0740-9223-b358-712a-18362be9b82f.png)  
  
６．「デバッグ」⇒「開始」を選択  
ALT+F5：次のブレークまで実行  
F10:ステップオーバ... 次の行を実行。次が別のストアドの場合、ストアドの中には入らない  
F11:ステップイン...次の行を実行。次が別のストアドならストアドの中に入る。  
![sql.png](/image/3f9f8da0-edea-259d-7cc9-b31d7777cb05.png)  
  
# 注意  
・リモート側またはクライアント側がHomeEditionの場合はWindows認証が上手くいかずに、デバッグが実行できない可能性がある。  
  
![sql.png](/image/064ce394-a034-bbd6-f98f-724ce842e883.png)  
  
``  
T-SQLデバッグを開始できません。コンピュータ'XXXX'に接続できませんでした。Visual Studio リモート デバッガーは、このWindowsエディションをサポートしていません。  
``  
  
この原因はリモートデバッグにWindows認証が必要で、HomeEditionにはそれがふくまれていないため。  
http://social.msdn.microsoft.com/Forums/vstudio/en-US/77689d96-dc35-4a10-96e8-a0f98b33901c/visual-studio-2008-remote-debugging-and-windows-xp-sp2-home-edition?forum=vsdebug  
  
  
・上記のエラーが発生した場合で、SQLSERVERのあるマシンにMicrosoft SQL Server Management Studioを入れているのなら「localhost\SQLEXPRESS」というようにlocalhostと明示しておけばデバッグが可能。  
  
・SQLSERVER2005以前に対しては、Microsoft SQL Server Management Studioからはデバッグできない。VisualStudio 2008 Professionalのサーバ接続のウィンドウで接続してデバッグする必要がある。  
