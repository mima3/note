SQLServerはSQL　Server Management Studioという強力なGUIツールが提供されているため忘れがちになるが、スクリプトからSQLサーバーについての様々な情報を取得することができる。  
  
# SQLServerのメタ情報の取得方法  
## オブジェクト カタログ ビューの説明  
ここではSQLSERVERのオブジェクトの情報を格納しているオブジェクト カタログ ビューについて説明をする。このビューをSQLで取得することにより、様々なオブジェクトの情報を閲覧できる。  
  
|テーブル名|説明|  
|:---------|:---|  
|sys.objects|データベース内で作成されるユーザー定義のスキーマ スコープ オブジェクトごとに 1 行のデータを格納する<BR>http://msdn.microsoft.com/ja-jp/library/ms190324.aspx|  
|sys.columns| ビューやテーブルなど、列を持つオブジェクトの列ごとに 1 行のデータを返す。 <BR>http://msdn.microsoft.com/ja-jp/library/ms176106.aspx|  
|sys.types|システム型とユーザー定義の型ごとに 1 行のデータを格納<BR>http://msdn.microsoft.com/ja-jp/library/ms188021.aspx|  
|sys.indexes|テーブル、ビュー、テーブル値関数など、テーブル オブジェクトのインデックスまたはヒープごとに 1 行のデータを格納。<BR>http://msdn.microsoft.com/ja-jp/library/ms173760.aspx|  
|sys.index_columns|sys.indexes インデックスまたは順序付けられていないテーブル (ヒープ) の一部である列ごとに 1 つの行を含む<BR>http://msdn.microsoft.com/ja-jp/library/ms175105.aspx|  
|sys.triggers|TR トリガーまたは TA トリガーであるオブジェクトごとに、1 行のデータを格納。<BR>http://msdn.microsoft.com/ja-jp/library/ms188746.aspx|  
|sys.parameters|パラメーターを受け入れるオブジェクトのパラメーターごとに 1 行のデータを保持する。 オブジェクトがスカラー関数の場合、戻り値を説明する単一行も含まれる。 この行には parameter_id 値 0 が設定される<BR>http://msdn.microsoft.com/ja-jp/library/ms176074.aspx|  
  
## 情報スキーマービュー  
ISO 標準定義の INFORMATION_SCHEMA に従って、内部の情報を返すビュー。  
  
|テーブル名|説明|  
|:---------|:---|  
|INFORMATION_SCHEMA.TABLES|現在のユーザーが権限を持つ、現在のデータベース内のテーブルごとに 1 行のデータを返す。<BR>http://msdn.microsoft.com/ja-jp/library/ms186224.aspx|  
|INFORMATION_SCHEMA.COLUMNS|現在のデータベースの現在のユーザーがアクセスできる列ごとに 1 行のデータを返す<BR>http://msdn.microsoft.com/ja-jp/library/ms188348.aspx|  
|INFORMATION_SCHEMA.ROUTINES|現在のデータベースの現在のユーザーがアクセスできるストアド プロシージャと関数ごとに、1 行のデータを返す<BR>現在のデータベース内の、現在のユーザーがアクセスできるユーザー定義の関数またはストアド プロシージャのパラメーターごとに 1 行のデータを返す.|http://msdn.microsoft.com/ja-jp/library/ms188757.aspx|  
|INFORMATION_SCHEMA.PARAMETERS|http://msdn.microsoft.com/ja-jp/library/ms173796.aspx|  
  
## メタ情報取得のための関数やプロシージャ  
|関数名|説明|  
|:-----|:---|  
|OBJECT_NAME ( object_id )|オブジェクトIDからオブジェクト名に変換する関数|  
|OBJECT_ID(object_name)|オブジェクト名からオブジェクトIDを取得する関数|  
|OBJECT_DEFINITION(object_id )|指定したオブジェクトの定義の Transact-SQL ソース テキストを返す。たとえばストアドのオブジェクトIDを指定するとその内容が取得できる|  
| EXEC sp_depends 'dbo.テーブル名'|テーブル名に依存する関数をすべてぬきだす|  
  
# 使用例  
これまでに説明したビューやテーブルを操作して各種のメタ情報を取得するサンプルを以下に示す。  
すべてSQLで実行できるので、任意のスクリプト言語でSQLを実行すればよい。  
  
以下はVBScriptで行った例は下記に示す。  
https://github.com/mima3/SqlServerScript  
  
  
## ストアドプロシージャと関数の内容を取得するサンプル  
  
```sql
SELECT specific_name,object_definition(object_id(specific_name)) FROM INFORMATION_SCHEMA.ROUTINES
```  
  
## トリガーの一覧と、その内容を取得するサンプル:  
  
```sql
SELECT name,object_definition( object_id) FROM sys.triggers
```  
  
## プロシージャのパラメータを取得するサンプル：  
  
```sql
select 
    sys.objects.name,
    sys.parameters.name, 
	sys.parameters.user_type_id,
	systypes.name 
  from
    sys.objects
	INNER JOIN sys.parameters ON sys.parameters.object_id = sys.objects.object_id
    INNER JOIN systypes ON sys.parameters.user_type_id = systypes.xtype
  WHERE sys.objects.type in ('TR','IF','P','FN')
  ORDER BY sys.objects.name, sys.parameters.parameter_id
```  
## テーブルの列情報を表示する例  
  
```sql
SELECT 
  TABLE_NAME, 
  COLUMN_NAME,
  DATA_TYPE, 
  COLUMN_DEFAULT
FROM 
  INFORMATION_SCHEMA.COLUMNS
ORDER BY
  TABLE_NAME, ORDINAL_POSITION
```  
  
## インデックスのあるテーブルと列を表示する例  
  
```sql
  select 
    sys.indexes.name AS INDEX_NAME,  
    sys.objects.name AS TABLE_NAME,
	sys.columns.name AS COLUMN_NAME 
  from 
    sys.indexes 
	inner join sys.index_columns on sys.indexes.object_id = sys.index_columns.object_id
	inner join sys.columns on sys.columns.column_id = sys.index_columns.column_id 
	       and sys.columns.object_id = sys.index_columns.object_id
	inner join sys.objects on sys.indexes.object_id = sys.objects.object_id
  where
    sys.objects.type = 'U'
  order by sys.indexes.object_id, sys.indexes.name,sys.index_columns.column_id
```  
  
## 特定のテーブルに依存しているオブジェクトを表示する例  
  
```sql
EXEC sp_depends 'dbo.Table_1'
```  
  
# まとめ  
SQLServerのメタ情報はSQLで取得できる。このことは、簡単なスクリプト言語でそれらが扱えることを意味する。  
これを利用することで、様々なことが可能になる。  
たとえば、２つのデータベースに対してスキーマーやプロシージャが一致するか検査するスクリプトを記述したり、テーブルのスキーマ情報をWiki形式に出力して、RedmineやTracに登録したりすることができる。  
  
