Pythonで主だったデータベースを操作する方法を記述する。  
Pythonは2.xと3.x両方でWindows環境で実行している。  
  
また、テストデータは下記のページのT01Prefecture.zipを解凍してテーブルとデータを使うものとする。  
  
 __PHPプログラミング初心者入門講座__   
http://php5.seesaa.net/article/61269550.html  
  
# MySQL  
## 環境  
Python2.7 または3.3  
MySQL 5.6.15  
  
  
## データベースの設定  
  
 __テーブル__   
  
```sql
CREATE TABLE `t01prefecture` (
  `PREF_CD` int(3) NOT NULL DEFAULT '0',
  `PREF_NAME` varchar(10) DEFAULT NULL,
  PRIMARY KEY (`PREF_CD`)
) ENGINE=InnoDB D
```  
  
 __単一のレコードセットを返すストアド__   
  
```sql
DELIMITER $$
CREATE DEFINER=`username`@`%` PROCEDURE `test_sp`(IN fromNo INT,
                                                  IN toNo INT)
BEGIN
    select * from t01prefecture WHERE PREF_CD BETWEEN fromNo AND toNo ;
END$$
DELIMITER ;
```  
  
 __複数のレコードセットを返すストアド__   
  
```sql
DELIMITER $$
CREATE DEFINER=`username`@`%` PROCEDURE `test_sp2`(IN cd1 INT,IN cd2 INT)
BEGIN
  select * from t01prefecture WHERE PREF_CD = cd1;
  select * from t01prefecture WHERE PREF_CD = cd2;

END$$
DELIMITER ;
```  
  
__Functionの例__   
  
```
DELIMITER $$
CREATE DEFINER=`username`@`%` FUNCTION `test_fn`(cd INT) RETURNS varchar(10) CHARSET utf8
BEGIN
    DECLARE ret VARCHAR(10);
    SELECT PREF_NAME INTO ret  FROM t01prefecture WHERE PREF_CD = cd;
   
RETURN ret;
END$$
DELIMITER ;
```  
  
## Pythonのコード  
mysql-connector-python　を使用する  
http://dev.mysql.com/downloads/connector/python/  
  
```py
# -*- coding: cp932 -*-
# mysqlの操作サンプル
# easy_install mysql-connector-python
import mysql.connector

try:
    cnn = mysql.connector.connect(host='localhost',
                                  port=3306,
                                  db='Sample001',
                                  user='root',
                                  passwd='root',
                                  charset="cp932")
    cur = cnn.cursor()

    #試験データの整理
    pref_cd = 100
    cur.execute("""DELETE FROM t01prefecture WHERE PREF_CD >= %s""" , (pref_cd,))
    cnn.commit()

    print("単純なSELECT文==========================")
    from_id = 45
    to_id = 999

    # 以下は環境の文字コードにあわせること！
    cur.execute("""SELECT PREF_CD,PREF_NAME FROM t01prefecture
                WHERE PREF_CD BETWEEN %s AND %s""" , (from_id, to_id, ))
    rows = cur.fetchall()
    for row in rows:
        print("%d %s" % (row[0], row[1]))

    print("コミットの試験==========================")
    pref_cd = 100
    pref_name = "モテモテ王国"
    cur.execute("""INSERT INTO t01prefecture(PREF_CD, PREF_NAME)
                VALUES (%s, %s)""" , (pref_cd, pref_name))

    pref_cd = 101
    pref_name = "野望の国"
    cur.execute("""INSERT INTO t01prefecture(PREF_CD,PREF_NAME)
                VALUES (%s, %s)""" , (pref_cd, pref_name,))
    cnn.commit()

    print("ロールバックの試験==========================")
    pref_cd = 102
    pref_name = "ロールバック"
    cur.execute("""INSERT INTO t01prefecture(PREF_CD,PREF_NAME)
                VALUES (%s, %s)""" , (pref_cd, pref_name,))
    cnn.rollback()

    print("ストアドプロシージャの試験==========================")
    cur.callproc("test_sp", (from_id, to_id))
    for rs in cur.stored_results():
        print("レコードセット...")
        rows = rs.fetchall()
        for row in rows:
            print ("%d %s" % (row[0], row[1]))

    print("ストアドプロシージャの試験(複数）==================")
    cur.callproc("test_sp2", (1, 100))
    for rs in cur.stored_results():
        print("レコードセット...")
        rows = rs.fetchall()
        for row in rows:
            print ("%d %s" % (row[0], row[1]))

    print("ファンクションの試験==========================")
    pref_cd = 100
    cur.execute("""SELECT test_fn(%s)""" , (pref_cd,))
    rows = cur.fetchall()
    for row in rows:
        print("code:%d name:%s" % (pref_cd, row[0]))

    cur.close()
    cnn.close()
except (mysql.connector.errors.ProgrammingError) as e:
    print (e)
```  
  
## 使用感  
・MYSQLのストアドはSQLSERVERに近い感じ。  
  
・ストアドを実行したあとは、 cur.stored_resultsにレコードセットが入っている。。  
  
・MySQL-pythonってライブラリもあるが3.x系で動作しない。使い方はだいたい同じ。  
http://sourceforge.net/projects/mysql-python/  
  
  
# SQLSERVER  
  
## 環境  
Python2.7 or 3.3  
SQLSERVER EXPRESS 2012  
  
  
## データベースの設定  
SQL SERVER接続を許可する。  
![無題.png](/image/9e123023-df0a-d4fc-5d04-2cde9ee91506.png)  
  
  
 __テーブル__   
  
```sql
CREATE TABLE [dbo].[T01Prefecture](
    [PREF_CD] [int] NOT NULL,
    [PREF_NAME] [varchar](10) NULL,
PRIMARY KEY CLUSTERED
(
    [PREF_CD] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
) ON [PRIMARY]
```  
  
  
 __単一のレコードセットをかえすストアド__   
  
```sql 
CREATE PROCEDURE [dbo].[test_sp]
    @from INT,
    @to INT
AS
BEGIN
    -- SET NOCOUNT ON added to prevent extra result sets from
    -- interfering with SELECT statements.
    SET NOCOUNT ON;

    -- Insert statements for procedure here
    SELECT * FROM T01Prefecture WHERE PREF_CD BETWEEN @from AND @to
END
```  
  
 __複数のレコードセットをかえすストアド__   
  
```sql

CREATE PROCEDURE [dbo].[test_sp2]
    @cd1 INT,
    @cd2 INT
AS
BEGIN
    -- SET NOCOUNT ON added to prevent extra result sets from
    -- interfering with SELECT statements.
    SET NOCOUNT ON;

    -- Insert statements for procedure here
    SELECT * FROM T01Prefecture WHERE PREF_CD = @cd1
    SELECT * FROM T01Prefecture WHERE PREF_CD = @cd2
END
```  
  
## Pythonのコード　  
### SQLSERVERを直接操作  
 __pymssql__  
http://pymssql.org/en/latest/  
  
動作が不安定  
Python2.7(32bit) では一応動作する。  
Python3.3(64bit) ではインストールすらできない。  
  
```py
# -*- coding: cp932 -*-
# mssqlの操作サンプル
# pymssqlはVARCHARのエンコードが上手くいかないのでやめとく。
# easy_install pymssql

import pymssql


cnn = pymssql.connect(host="127.0.0.1\SQLEXPRESS", user="sa", password="xxxx", database="Sample001")
cur = cnn.cursor()

# 試験データの整理
pref_cd = 100
cur.execute("""DELETE FROM t01prefecture WHERE PREF_CD >= %d"""
            % (pref_cd,))
cnn.commit()

print("単純なSELECT文==========================")
print("VARCHARのエンコードがおかしいことを確認")
from_id = 45
to_id = 999
cur.execute("""SELECT PREF_CD, PREF_NAME FROM T01Prefecture
            WHERE PREF_CD BETWEEN %d AND %d""" , (from_id, to_id))
rows = cur.fetchall()
for row in rows:
    print(row)
    # print("%d %s" % (row[0], row[1])) #Errorになる
    # VARCHARを取り扱った場合の文字コードの解析が不正
    # CP932の文字コードをUNICODEとして扱おうとしている。

print("VARCHARのフィールドは扱えないのでNVARCHARに変換する")
# VARCHARのフィールドは扱えないのでNVARCHARに変換して返す
cur.execute("""SELECT PREF_CD,CAST(PREF_NAME  AS NVARCHAR) FROM T01Prefecture
            WHERE PREF_CD BETWEEN %d AND %d""" , (from_id, to_id))
rows = cur.fetchall()
for row in rows:
    print(row)
    print("%d %s" % (row[0], row[1]))

print("コミットの試験==========================")
pref_cd = 100
pref_name = "モテモテ国"
cur.execute("""INSERT INTO t01prefecture(PREF_CD, PREF_NAME)
            VALUES (%d, %s)""" , (pref_cd, pref_name))

pref_cd = 101
pref_name = "野望の国"
cur.execute("""INSERT INTO t01prefecture(PREF_CD,PREF_NAME)
            VALUES (%d, %s)""" , (pref_cd, pref_name,))
cnn.commit()

print("ロールバックの試験==========================")
pref_cd = 102
pref_name = "ロール"
cur.execute("""INSERT INTO t01prefecture(PREF_CD,PREF_NAME)
            VALUES (%d, %s)""" , (pref_cd, pref_name,))
cnn.rollback()


cur.execute("""SELECT PREF_CD, CAST(PREF_NAME AS NVARCHAR) FROM t01prefecture
            WHERE PREF_CD BETWEEN %d AND %d""" , (from_id, to_id, ))
rows = cur.fetchall()
for row in rows:
    print("%d %s" % (row[0], row[1]))

print("ストアドプロシージャの試験==========================")
cur.execute("""exec test_sp %d, %d """ , (from_id, to_id, ))
rows = cur.fetchall()
for row in rows:
    #エンコードが不正
    #print("%d %s" % (row[0], row[1]))
    print(row)

print("ストアドプロシージャの試験 複数==========================")
cur.execute("""exec test_sp2 %d, %d """ , (1, 10, ))
for row in rows:
    while True:
        print ("レコードセット...")
        rows = cur.fetchall()
        for row in rows:
            #print("%d %s" % (row[0], row[1]))
            print(row)
        if not cur.nextset():
            break

cur.close()
cnn.close()

```  
  
  
### ODBC経由  
 __pyodbcの使用：__   
https://code.google.com/p/pyodbc/  
  
```py
# -*- coding: cp932 -*-
# mssqlの操作サンプル
# easy_install pyodbc

import pyodbc

try:
    cnn = pyodbc.connect("DRIVER={SQL Server};SERVER=127.0.0.1\SQLEXPRESS;" +
                         "UID=sa;PWD=XXXX;DATABASE=Sample001")
    cur = cnn.cursor()
    #試験データの整理
    pref_cd = 100
    cur.execute("""DELETE FROM t01prefecture WHERE PREF_CD >= ?""", pref_cd)
    cnn.commit()

    print("単純なSELECT文==========================")
    from_id = 45
    to_id = 999
    # 以下は環境の文字コードにあわせること！
    cur.execute("""SELECT PREF_CD,PREF_NAME FROM t01prefecture
                WHERE PREF_CD BETWEEN ? AND ?""" , from_id, to_id)
    rows = cur.fetchall()
    for row in rows:
        print("%d %s" % (row[0], row[1]))

    print("コミットの試験==========================")
    pref_cd = 100
    pref_name = "モテモテ国"
    cur.execute("""INSERT INTO t01prefecture(PREF_CD, PREF_NAME)
                VALUES (?, ?)""" , pref_cd, pref_name)

    pref_cd = 101
    pref_name = "野望の国"
    cur.execute("""INSERT INTO t01prefecture(PREF_CD,PREF_NAME)
                VALUES (?, ?)""" , pref_cd, pref_name)
    cnn.commit()

    print("ロールバックの試験==========================")
    pref_cd = 102
    pref_name = "ロール"
    cur.execute("""INSERT INTO t01prefecture(PREF_CD,PREF_NAME)
                VALUES (?, ?)""" , pref_cd, pref_name)
    cnn.rollback()

    cur.execute("""SELECT PREF_CD,PREF_NAME FROM t01prefecture
                WHERE PREF_CD BETWEEN ? AND ?""" , from_id, to_id)
    rows = cur.fetchall()
    for row in rows:
        print("%d %s" % (row[0], row[1]))

    print("ストアドプロシージャの試験==========================")
    cur.execute("""exec test_sp ?, ? """ , from_id, to_id)
    rows = cur.fetchall()
    for row in rows:
        print("%d %s" % (row[0], row[1]))

    print("ストアドプロシージャの試験 複数 ====================")
    cur.execute("""exec test_sp2 ?, ? """ , 1, 10)
    while True:
        print ("レコードセット...")
        rows = cur.fetchall()
        for row in rows:
            print("%d %s" % (row[0], row[1]))
        if not cur.nextset():
            break

    cur.close()
    cnn.close()

except (pyodbc.Error) as e:
    print (e)
    print (e.args[1])

```  
  
 __pypyodbcの使用例：__   
https://code.google.com/p/pypyodbc/  
  
```py
# -*- coding: cp932 -*-
# mssqlの操作サンプル
# easy_install pypyodbc

import pypyodbc

try:
    cnn = pypyodbc.connect("DRIVER={SQL Server};SERVER=127.0.0.1\SQLEXPRESS;UID=sa;PWD=xxxxx;DATABASE=Sample001")
    cur = cnn.cursor()

    #試験データの整理
    pref_cd = 100
    cur.execute("""DELETE FROM t01prefecture WHERE PREF_CD >= ?"""
                , (pref_cd,))
    cnn.commit()

    print("単純なSELECT文==========================")
    from_id = 45
    to_id = 999
    # 以下は環境の文字コードにあわせること！
    cur.execute("""SELECT PREF_CD,PREF_NAME FROM t01prefecture
                WHERE PREF_CD BETWEEN ? AND ?""" , (from_id, to_id, ))
    rows = cur.fetchall()
    for row in rows:
        print("%d %s" % (row[0], row[1]))


    print("コミットの試験==========================")
    pref_cd = 100
    pref_name = "モテモテ国"
    cur.execute("""INSERT INTO t01prefecture(PREF_CD, PREF_NAME)
                VALUES (?, ?)""" , (pref_cd, pref_name))

    pref_cd = 101
    pref_name = "野望の国"
    cur.execute("""INSERT INTO t01prefecture(PREF_CD,PREF_NAME)
                VALUES (?, ?)""" , (pref_cd, pref_name,))
    cnn.commit()

    print("ロールバックの試験==========================")
    pref_cd = 102
    pref_name = "ロール"
    cur.execute("""INSERT INTO t01prefecture(PREF_CD,PREF_NAME)
                VALUES (?, ?)""" , (pref_cd, pref_name,))
    cnn.rollback()


    cur.execute("""SELECT PREF_CD,PREF_NAME FROM t01prefecture
                WHERE PREF_CD BETWEEN ? AND ?""" , (from_id, to_id, ))
    rows = cur.fetchall()
    for row in rows:
        print("%d %s" % (row[0], row[1]))

    print("ストアドプロシージャの試験==========================")
    cur.execute("""exec test_sp ?, ? """ , (from_id, to_id, ))
    rows = cur.fetchall()
    for row in rows:
        print("%d %s" % (row[0], row[1]))

    print("ストアドプロシージャの試験 複数 ==========================")
    cur.execute("""exec test_sp2 ?, ? """ , (1, 10, ))
    while True:
        print ("レコードセット...")
        rows = cur.fetchall()
        for row in rows:
            print("%d %s" % (row[0], row[1]))
        if not cur.nextset():
            break

    cur.close()
    cnn.close()
except (pypyodbc.DatabaseError) as e:
    print (e.args[1])
 
```  
  
## 使用感  
pymssqlはODBCを経由せずに使用できるが、動作が極めて不安定。  
VARCHARの挙動があやしくて、NVARCHARにキャストしないと使用できない。  
また、64bitのPython3.3ではインストールすらできない。  
  
ODBC経由のライブラリはいずれも動作した。  
Windows以外で確認はしていないが、サポートはしているとのこと。  
  
# ORACLE  
## 環境  
Python 2.7 または 3.3  
Oracle11  
1.Oracleクライアントをインストールする。この際、開発者用の環境をつくる。  
　  
2.以下からダウンロードしてインストールするか、easy_installする。  
   cx_Oracle  
   http://cx-oracle.sourceforge.net/  
  
## Pythonのコード  
  
```py
# -*- coding: cp932 -*-
# cx_Oracleを用いたPythonでのORACLE操作
# 1.Oracleクライアントをインストールする。
# 　この際、開発者用の環境をつくる。
# 　（おそらく、OCIが必要？）
# 　
# 2.以下からダウンロードしてインストールするか、easy_installする。
#  cx_Oracle
#  http://cx-oracle.sourceforge.net/
import cx_Oracle
import os
os.environ["NLS_LANG"] = "JAPANESE_JAPAN.JA16SJISTILDE"
try:
    tns = cx_Oracle.makedsn("localhost", 1521, "Sample")
    conn = cx_Oracle.connect("user", "pass", tns)
    cur = conn.cursor()
    from_cd = 45
    to_cd = 200
    cur.execute("""SELECT * FROM T01PREFECTURE
                WHERE PREF_CD BETWEEN :arg1 AND :arg2""",
                arg1=from_cd,
                arg2=to_cd)
    rows = cur.fetchall()
    for r in rows:
        print("%d : %s" % (r[0], r[1]))

    print ("===================")
    print ("100を消す")
    cur.execute("""DELETE FROM T01PREFECTURE WHERE PREF_CD=:arg1""",
                arg1=100)

    cur.execute("""SELECT * FROM T01PREFECTURE
                WHERE PREF_CD BETWEEN :arg1 AND :arg2""",
                arg1=from_cd,
                arg2=to_cd)
    rows = cur.fetchall()
    for r in rows:
        print("%d : %s" % (r[0], r[1]))

    print ("------------------")
    print ("100 を追加")
    cur.execute("""INSERT INTO T01PREFECTURE
                VALUES (:arg1, :arg2)""",
                arg1=100,
                arg2="あたた")
    conn.commit()

    cur.execute("""SELECT * FROM T01PREFECTURE
                WHERE PREF_CD BETWEEN :arg1 AND :arg2""",
                arg1=from_cd,
                arg2=to_cd)
    rows = cur.fetchall()
    for r in rows:
        print("%d : %s" % (r[0], r[1]))

    print ("===================")
    print ("101追加")
    cur.execute("""INSERT INTO T01PREFECTURE
                VALUES (:arg1, :arg2)""",
                arg1=101,
                arg2="北斗")

    cur.execute("""SELECT * FROM T01PREFECTURE
                WHERE PREF_CD BETWEEN :arg1 AND :arg2""",
                arg1=from_cd,
                arg2=to_cd)
    rows = cur.fetchall()
    for r in rows:
        print("%d : %s" % (r[0], r[1]))

    print ("------------------")
    print ("ロールバック")
    conn.rollback()
    cur.execute("""SELECT * FROM T01PREFECTURE
                WHERE PREF_CD BETWEEN :arg1 AND :arg2""",
                arg1=from_cd,
                arg2=to_cd)
    rows = cur.fetchall()
    for r in rows:
        print("%d : %s" % (r[0], r[1]))

except (cx_Oracle.DatabaseError) as ex:
    error, = ex.args
    print (error.message)
```  
  
## 使用感  
OCI経由で操作しているようなので、ORACLEクライアントをインストールせねばならない。  
接続方法に癖がある。あと文字コードの指定は環境変数でおこなう。  
  
ORACLEのPL/SQLはSQLSERVERと違って結果セット返さない。  
（配列を返せるが、ここでは面倒なので実験していない）  
  
明示的にCOMMITしないと変更は破棄される。  
  
# Postgresql  
  
## 環境  
Python 2.x / 3.x  
PostgresSQL 9.3  
  
ライブラリとしてpsycopg2をつかう  
Windowsの場合は以下のページからダウンロードしてExeを実行  
http://www.stickpeople.com/projects/python/win-psycopg/  
  
## データベースの設定  
  
 __テーブル定義__  
  
```sql
CREATE TABLE t01prefecture
(
  pref_cd integer NOT NULL,
  pref_name character varying(10),
  CONSTRAINT t01prefecture_pkey PRIMARY KEY (pref_cd)
)
WITH (
  OIDS=FALSE
);
ALTER TABLE t01prefecture
  OWNER TO postgres;
```  
  
  
  
 __PostgresSQLのユーザ定義関数__   
  
```sql
CREATE OR REPLACE FUNCTION test_sp(IN from_cd integer, IN to_cd integer)
  RETURNS TABLE(code integer, name varchar) AS
$$
DECLARE
BEGIN
    RETURN QUERY SELECT PREF_CD,PREF_NAME FROM t01Prefecture
            WHERE PREF_CD BETWEEN from_cd AND to_cd;
END;
$$ LANGUAGE plpgsql;
```  
  
## Pythonのコード  
  
```py
# -*- coding: cp932 -*-
# Winddows の場合は以下からダウンロード
# 　http://www.stickpeople.com/projects/python/win-psycopg/
# Python3.xの場合、unicode(row[1],'utf-8') は不要。
#
import psycopg2

try:
    cnn = psycopg2.connect("dbname=Sample001 host=localhost user=postgres password=xxxxx")
    cur = cnn.cursor()

    #試験データの整理
    pref_cd = 100
    cur.execute("""DELETE FROM t01prefecture WHERE PREF_CD >= %s"""
                , (pref_cd,))
    cnn.commit()

    print("単純なSELECT文==========================")
    from_id = 45
    to_id = 999
    cur.execute("""SELECT PREF_CD,PREF_NAME FROM t01prefecture
                WHERE PREF_CD BETWEEN %s AND %s""" , (from_id, to_id, ))
    rows = cur.fetchall()
    for row in rows:
        #print("%d %s" % (row[0], unicode(row[1],'utf-8')))
        print("%d %s" % (row[0], row[1]))

    print("コミットの試験==========================")
    pref_cd = 100
    pref_name = u"モテモテ国"
    cur.execute(u"""INSERT INTO t01prefecture(PREF_CD, PREF_NAME)
                VALUES (%s, %s)""" , (pref_cd, pref_name))

    pref_cd = 101
    pref_name = u"野望の国"
    cur.execute(u"""INSERT INTO t01prefecture(PREF_CD,PREF_NAME)
                VALUES (%s, %s)""" , (pref_cd, pref_name,))
    cnn.commit()
    cur.execute("""SELECT PREF_CD,PREF_NAME FROM t01prefecture
                WHERE PREF_CD BETWEEN %s AND %s""" , (from_id, to_id, ))
    rows = cur.fetchall()
    for row in rows:
        #print("%d %s" % (row[0],unicode(row[1],'utf-8')))
        print("%d %s" % (row[0],row[1]))


    print("ロールバックの試験==========================")
    pref_cd = 102
    pref_name = u"ロール"
    cur.execute(u"""INSERT INTO t01prefecture(PREF_CD,PREF_NAME)
                VALUES (%s, %s)""" , (pref_cd, pref_name,))

    cur.execute("""SELECT PREF_CD,PREF_NAME FROM t01prefecture
                WHERE PREF_CD BETWEEN %s AND %s""" , (from_id, to_id, ))
    rows = cur.fetchall()
    for row in rows:
        #print("%d %s" % (row[0], unicode(row[1],'utf-8')))
        print("%d %s" % (row[0], row[1]))

    print("-------------------------")
    cnn.rollback()
    cur.execute("""SELECT PREF_CD,PREF_NAME FROM t01prefecture
                WHERE PREF_CD BETWEEN %s AND %s""" , (from_id, to_id, ))
    rows = cur.fetchall()
    for row in rows:
        #print("%d %s" % (row[0], unicode(row[1],'utf-8')))
        print("%d %s" % (row[0], row[1]))

    print("ユーザー定義==========================")
    cur.execute("""SELECT * FROM test_sp(%s,%s)""" , (from_id, to_id, ))
    rows = cur.fetchall()
    for row in rows:
        #print("%d %s" % (row[0], unicode(row[1],'utf-8')))
        print("%d %s" % (row[0], row[1]))

    cur.close()
    cnn.close()

except (psycopg2.OperationalError) as e:
    print (e)

```  
  
## 使用感  
Python2.XはレコードセットがUTF8で帰ってくるので、一旦UNICODEに変換してやらねばならない。  
Python3系は不要。  
  
PostgresのストアドプロシージャーはORACLEのPL/SQLに近い。  
ただ、テーブル型を戻り値にする関数を指定できる。  
  
# SQLite  
  
## 環境  
特記事項なし。  
Python2.5以降なら標準でSQLITE3が操作できるはず。  
  
## Pythonのコード  
  
```py
# -*- coding: cp932 -*-
# sqlite3はPython2.5から以降から標準であるはず.
import sqlite3
conn = sqlite3.connect('test.sqlite3')
sql = '''CREATE TABLE  IF NOT EXISTS  t01prefecture(
                         pref_cd INTEGER,
                         pref_name TEXT);'''
conn.execute(sql)

conn.execute(u"DELETE FROM t01prefecture")

# コミットの試験
pref_cd = 100
pref_name = u"モテモテ国"
conn.execute(u"""INSERT INTO t01prefecture(PREF_CD, PREF_NAME)
            VALUES (?, ?)""" , (pref_cd, pref_name))

pref_cd = 101
pref_name = u"野望の国"
conn.execute(u"""INSERT INTO t01prefecture(PREF_CD,PREF_NAME)
            VALUES (?, ?)""" , (pref_cd, pref_name,))
conn.commit()

# ロールバックの試験
pref_cd = 102
pref_name = u"back"
conn.execute(u"""INSERT INTO t01prefecture(PREF_CD,PREF_NAME)
            VALUES (?, ?)""" , (pref_cd, pref_name,))
conn.rollback()

rows = conn.execute(u'SELECT * FROM t01prefecture WHERE pref_cd > ?', (0,))
for row in rows:
    print(u"%d %s" % (row[0], row[1]))

# ユーザ定義
# 文字を連結するのみ
class UserDef:
    def __init__(self):
        self.values = []
    def step(self, value):
        self.values.append(value)
    def finalize(self):
        return "/".join(map(str, self.values)) 

conn.create_aggregate("userdef", 1, UserDef)
rows = conn.execute(u'SELECT userdef(PREF_NAME) FROM t01prefecture')
for row in rows:
    print(u"%s" % (row[0]))

conn.close()
```  
  
## 使用感  
いままでのDBと違い、サーバーを構築する必要がない。  
ユーザ定義の関数をクライアント側で設定できる。  
  
