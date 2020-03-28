## 背景  
SQLiteをPythonから使用する場合、Python2.5以上ではデフォルトでsqlite3が使用できます。  
  
これは、DB-API 2.0 仕様に準拠した SQL インタフェースを提供するもので、pysqliteという名前で開発されています。  
  
 **sqlite3**   
http://docs.python.jp/2/library/sqlite3.html  
  
しかしながら、このライブラリはSQLiteの全ての機能を使えるわけではありません。これは、DB-APIの仕様に準拠するためです。  
  
APSWは、SQLite APIをPythonで完全に使用できるようなライブラリになっています。APSWはPython2.xとPython3.xで動作します。  
  
https://github.com/rogerbinns/apsw  
http://rogerbinns.github.io/apsw/  
  
## インストール方法  
pipやeasy_installではインストールが行えません。  
  
**2019.08.03 更新**  
現在は以下でもインストール可能です。  
  
```
pip install --user https://github.com/rogerbinns/apsw/releases/download/3.28.0-r1/apsw-3.28.0-r1.zip --global-option=fetch --global-option=--version --global-option=3.28.0 --global-option=--all --global-option=build --global-option=--enable-all-extensions
```  
  
最新情報は下記を参照してください。  
https://rogerbinns.github.io/apsw/download.html#easy-install-pip-pypi  
  
  
### Windowsの場合  
以下から該当するバイナリをダウンロードしてインストーラーを実行します。  
  
http://rogerbinns.github.io/apsw/download.html#source-and-binaries  
  
VisualStudioを持っている場合、UNIXと同様にソースからのビルドが可能です。  
（mingw32だと、ビルドはできても実行時にDLLが足りなくて動作しませんでした。）  
  
### Unixの場合  
ソースコードをダウンロードして、下記のコマンドを実行してください。  
  
```
python setup.py fetch --all build --enable-all-extensions install
```  
  
ビルドについての詳細は下記のページに記述してあります。  
http://rogerbinns.github.io/apsw/build.html  
  
## APSWとpysqliteの違い  
APSWとpysqliteは根本的に異なる方向からSQLiteへのアクセス方法を提供します。  
  
APSWはSQLiteのバージョン３のみをラップしており、全てのAPIにアクセスできる方法を提供します。  
  
pysqliteはDBAPI準拠のラッパーを提供するために、他のデータベースと同じような振る舞いを行うようにします。そのため、いくつかのSQLiteの特徴を隠ぺいします。  
  
以下に、APSWの利点や拡張された機能を紹介します。  
  
### 最新のSQLiteの機能が使える  
APSWはSQLiteの最新が使用できるようにしています。もし、SQLiteに機能の追加や変更が行われたらAPSWもそれらの機能を追いかけます。  
  
### Virtual Tableが使用可能である。  
[Virtual Table](https://sqlite.org/vtab.html) はSQLite 3.3.7から導入された機能です。  
  
Virtual TableはSQL文の観点からは他のテーブルやビューと変わりありません。しかし、舞台裏ではVirtual Tableへのクエリや書き込みは、データベースファイルの読み書きの代わりにコールバックメソッドを引き起こします。  
  
以下に、二次元配列のデータをSQLで操作する例を示します。  
  
```py
# -*- coding: utf-8 -*- 
import os, sys, time
import apsw
connection=apsw.Connection(":memory:")
cursor=connection.cursor()

###
### Virtual tables
### 

#
data = [
    [1, 'test1', 'categoryA'],
    [2, 'test2', 'categoryA'],
    [3, 'test3', 'categoryA'],
    [4, 'test4', 'categoryB'],
    [5, 'test5', 'categoryB'],
    [6, 'test6', 'categoryB'],
    [7, 'test7', 'categoryB'],
    [8, 'test8', 'categoryC'],
    [9, 'test9', 'categoryC'],
    [10, 'test10', 'categoryC']
]
counter = len(data)


# This gets registered with the Connection
class Source:
    def Create(self, db, modulename, dbname, tablename, *args):
        columns = ['rowid', 'name', 'category']
        schema="create table foo("+','.join(["'%s'" % (x,) for x in columns[1:]])+")"
        return schema,Table(columns,data)
    Connect=Create

# Represents a table
class Table:
    def __init__(self, columns, data):
        self.columns=columns
        self.data=data

    def BestIndex(self, *args):
        return None

    def Open(self):
        return Cursor(self)

    def Disconnect(self):
        pass

    def UpdateChangeRow(self, row, newrowid, fields):
        for d in data:
            if(d[0] == row):
                d[0] = newrowid
                d[1] = fields[0]
                d[2] = fields[1]

    def UpdateDeleteRow(self, row):
        for i in range(len(data)):
            if(data[i][0] == row):
                del data[i]
                return

    def UpdateInsertRow(self, rowid, fields):
        global counter
        counter = counter + 1
        data.append([counter, fields[0], fields[1]])
        return counter

    Destroy=Disconnect

# Represents a cursor
class Cursor:
    def __init__(self, table):
        self.table=table

    def Filter(self, *args):
        self.pos=0

    def Eof(self):
        return self.pos>=len(self.table.data)

    def Rowid(self):
        return self.table.data[self.pos][0]

    def Column(self, col):
        return self.table.data[self.pos][1+col]

    def Next(self):
        self.pos+=1

    def Close(self):
        pass

connection.createmodule("source", Source())
cursor.execute("create virtual table test using source()")
ret = cursor.execute("select * from test where category = 'categoryB'")
for row in ret:
    print row[0], row[1]
print ('update -----------------')
cursor.execute("update test set category='categoryB' where name='test1'")
ret = cursor.execute("select * from test where category = 'categoryB'")
for row in ret:
    print row[0], row[1]

print ('delete -----------------')
cursor.execute("delete from test where name='test4'")
ret = cursor.execute("select * from test")
for row in ret:
    print row[0], row[1]

print ('insert ----------------')
cursor.execute("insert into test values('xxxx','yyyy')")
ret = cursor.execute("select * from test")
for row in ret:
    print row[0], row[1]

```  
  
このように、VirutalTableを用いることで任意のデータをSQLで操作することができます。公式のサンプルでは、ディレクトリ中のファイルをSQLで選択するサンプルがあります。  
http://apidoc.apsw.googlecode.com/hg/example.html  
  
その他、VirtualTableの詳細は下記を参照してください。  
http://rogerbinns.github.io/apsw/vtable.html  
  
### Virtual File System (VFS)が使用可能である  
SQLiteのコアおよび基礎となるオペレーティング·システム間のインタフェースを定義するVFSが使用できます。  
  
APSWでは、VFSの機能を利用することができ、また、既定のVFSを継承して拡張した機能を持たせることができます。たとえば、以下の例ではVFSを用いてSQLiteのファイルを難読化しています。  
  
```py
# -*- coding: utf-8 -*- 
import os, sys, time
import apsw

### このサンプルは以下から抜粋したもの
### http://apidoc.apsw.googlecode.com/hg/example.html
### VFSを使ってデータベースを難読化する
### スキーマのすべてのバイトを0xa5でXORする。
### この方式はMAPIとSQL SERVERで用いられる
###

def encryptme(data):
    if not data: return data
    return "".join([chr(ord(x)^0xa5) for x in data])

# ""の基底からの継承はデフォルトのVFSを意味する
class ObfuscatedVFS(apsw.VFS):
    def __init__(self, vfsname="obfu", basevfs=""):
        self.vfsname=vfsname
        self.basevfs=basevfs
        apsw.VFS.__init__(self, self.vfsname, self.basevfs)

    # 独自のファイルを実装したいが、またそれは継承させたい
    def xOpen(self, name, flags):
        # We can look at uri parameters
        if isinstance(name, apsw.URIFilename):
            print "fast is", name.uri_parameter("fast")
            print "level is", name.uri_int("level", 3)
            print "warp is", name.uri_boolean("warp", False)
            print "notpresent is", name.uri_parameter("notpresent")
        return ObfuscatedVFSFile(self.basevfs, name, flags)

# 暗号ルーチンを実装するためにxReadとxWriteをオーバーライドする
class ObfuscatedVFSFile(apsw.VFSFile):
    def __init__(self, inheritfromvfsname, filename, flags):
        apsw.VFSFile.__init__(self, inheritfromvfsname, filename, flags)

    def xRead(self, amount, offset):
        return encryptme(super(ObfuscatedVFSFile, self).xRead(amount, offset))

    def xWrite(self, data, offset):
        super(ObfuscatedVFSFile, self).xWrite(encryptme(data), offset)


# To register the VFS we just instantiate it
obfuvfs=ObfuscatedVFS()
# Lets see what vfs are now available?
print apsw.vfsnames()

# Make an obfuscated db, passing in some URI parameters
obfudb=apsw.Connection("file:myobfudb?fast=speed&level=7&warp=on",
                       flags=apsw.SQLITE_OPEN_READWRITE | apsw.SQLITE_OPEN_CREATE | apsw.SQLITE_OPEN_URI,
                       vfs=obfuvfs.vfsname)
# Check it works
obfudb.cursor().execute("create table foo(x,y); insert into foo values(1,2)")

# 実際のディスクの中身を確認する
print `open("myobfudb", "rb").read()[:20]`
# '\xf6\xf4\xe9\xcc\xd1\xc0\x85\xc3\xca\xd7\xc8\xc4\xd1\x85\x96\xa5\xa1\xa5\xa4\xa4'

print `encryptme(open("myobfudb", "rb").read()[:20])`
# 'SQLite format 3\x00\x04\x00\x01\x01'

# Tidy up
obfudb.close()
os.remove("myobfudb")
```  
  
詳細については下記を参照してください。  
http://rogerbinns.github.io/apsw/vfs.html  
  
### BLOB I/O が使用できる  
Blobは、バイトのシーケンスを表すSQLiteのデータ型です。それは、サイズがゼロ以上のバイトです。  
  
APSWを用いて、このBlobに対して読み書きが行えます。以下に、その使用例を示します。  
  
```py
# -*- coding: utf-8 -*- 
import os, sys, time
import apsw
import os
connection=apsw.Connection("blob.sqlite")
cursor=connection.cursor()

###
### Blob I/O
### http://apidoc.apsw.googlecode.com/hg/example.html

cursor.execute("create table blobby(x,y)")
# Add a blob we will fill in later
cursor.execute("insert into blobby values(1,zeroblob(10000))")
# Or as a binding
cursor.execute("insert into blobby values(2,?)", (apsw.zeroblob(20000),))
# Open a blob for writing.  We need to know the rowid
rowid=cursor.execute("select ROWID from blobby where x=1").next()[0]
blob=connection.blobopen("main", "blobby", "y", rowid, 1) # 1 is for read/write
blob.write("hello world")
blob.seek(100)
blob.write("hello world, again")
blob.close()

```  
  
詳細については下記を参考にしてください。  
http://rogerbinns.github.io/apsw/blob.html  
  
### バックアップが使用できる  
APSWではbackupを用いて接続中のDBを別の接続中のDBにバックアップが可能です。  
  
```py
# -*- coding: utf-8 -*- 
import os, sys, time
import apsw
import os
connection=apsw.Connection("src.sqlite")
cursor=connection.cursor()

# バックアップ元を作成
cursor.execute("create table test(x,y)")
cursor.execute("insert into test values(1,'TEST1')")
cursor.execute("insert into test values(2,'TEST2')")
cursor.execute("insert into test values(3,'TEST3')")
cursor.execute("insert into test values(4,'TEST4')")
cursor.execute("insert into test values(5,'TEST5')")

# バックアップ
memcon=apsw.Connection("backup.sqlite")
with memcon.backup("main", connection, "main") as backup:
    backup.step() # copy whole database in one go

for row in memcon.cursor().execute("select * from test"):
    print row[0], row[1]
    pass

```  
  
詳細については下記を参考にしてください。  
http://rogerbinns.github.io/apsw/backup.html  
  
### スレッドをまたいだ操作が可能  
connectionやcursorをスレッドをまたいで共有できます。  
pysqliteの場合、Connectionやcursorsは同じスレッドで使用しなければいけません。  
  
**:pysqliteの例**  
```py:pysqliteの例
# -*- coding: utf-8 -*- 
import threading
import sqlite3
def func(t):
    return 1 + t


class TestThread(threading.Thread):
    def __init__(self, conn):
        threading.Thread.__init__(self)
        self.conn = conn

    def run(self):
        self.conn.create_function("func", 1, func)
        cur = self.conn.cursor()
        ret = cur.execute("select func(3)")
        for row in ret:
            print(row[0])


conn = sqlite3.connect(":memory:")
th = TestThread(conn)
th.start()
th.join()

```  
  
pysqliteではスレッドをまたいだ操作が認められないので以下のような例外が発生します。  
  
```
Exception in thread Thread-1:
Traceback (most recent call last):
  File "C:\Python27\lib\threading.py", line 810, in __bootstrap_inner
    self.run()
  File "test_thread.py", line 14, in run
    self.conn.create_function("func", 1, func)
ProgrammingError: SQLite objects created in a thread can only be used in that sa
me thread.The object was created in thread id 19540 and this is thread id 4652
```  
  
APSWの場合、同様のスレッドをまたいでも例外は発生しません。  
  
**:APSWでスレッドをまたぐ**  
```py:APSWでスレッドをまたぐ
# -*- coding: utf-8 -*- 
import threading
import apsw


def func(t):
    return 1 + t


class TestThread(threading.Thread):
    def __init__(self, conn):
        threading.Thread.__init__(self)
        self.conn = conn

    def run(self):
        self.conn.createscalarfunction("func", func, 1)
        cur = self.conn.cursor()
        ret = cur.execute("select func(3)")
        for row in ret:
            print(row[0])


conn = apsw.Connection(":memory:")
th = TestThread(conn)
th.start()
th.join()
```  
  
ただし、排他処理を注意深くあつかわないとクラッシュやデッドロックを引き起こします。  
  
### ネストしたトランザクションの利用  
APSWではConnectionのContext Managerを使用することでネストしたトランザクションを使用できます。pysqliteでは1度に1つのトランザクションしか利用できず、ネストはできません。  
  
このネストしたトランザクションで用いるSavePointはSQLite3.6.8で追加されたものです。これはSQLiteを最新の状態で使用できるAPSWの利点の一つと言えるでしょう。  
  
以下にネストしたトランザクションの例を示します。  
  
```py
# -*- coding: utf-8 -*- 
import os, sys, time
import apsw
import os
connection=apsw.Connection(":memory:")

connection.cursor().execute("create table test(x primary key,y)")
with connection: # with でトランザクションを開始。例外ならRollback、それ以外はCommit
    connection.cursor().execute("insert into test values(1,'TEST1')")
    try: # with でSAVEPOINTを開始。例外ならRollback、それ以外はCommit
        with connection:
            connection.cursor().execute("insert into test values(2,'TEST2')")
            connection.cursor().execute("insert into test values(3,'TEST3')")
    except Exception, ex:
        print (ex)
        print ('rollback 1')
    try:
        with connection: # 以下のSQLはエラーがでて記録されない
            connection.cursor().execute("insert into test values(4,'TEST4')")
            connection.cursor().execute("insert into test values(4,'Error')")
    except Exception, ex:
        print (ex)
        print ('rollback 2')
    try:
        with connection:
            connection.cursor().execute("insert into test values(5,'TEST5')")
            connection.cursor().execute("insert into test values(6,'TEST6')")
    except Exception, ex:
        print (ex)
        print ('rollback 3')

for row in connection.cursor().execute("select * from test"):
    print row[0], row[1]

```  
  
 **ConnectionのContextについて**   
http://rogerbinns.github.io/apsw/connection.html#apsw.Connection.__enter__  
  
 **SQLiteのネストされたトランザクションについて**   
https://sqlite.org/lang_savepoint.html  
  
### 複数コマンドの実行  
APSWではセミコロンで区切ることにより、複数のコマンドが実行可能です。  
  
```py
# -*- coding: utf-8 -*- 
import apsw
con=apsw.Connection(":memory:")
cur=con.cursor()
for row in cur.execute("create table foo(x,y,z);insert into foo values (?,?,?);"
                       "insert into foo values(?,?,?);select * from foo;drop table foo;"
                       "create table bar(x,y);insert into bar values(?,?);"
                       "insert into bar values(?,?);select * from bar;",
                       (1,2,3,4,5,6,7,8,9,10)):
                           print row
```  
  
### SELECTのようなデータを返すSQLがCursor.executemany()で使用可能  
APSWではSELECTのようなデータを返すSQLがCursor.executemany()で使用可能になっています。  
  
```py
# -*- coding: utf-8 -*- 
import apsw
con=apsw.Connection(":memory:")
cur=con.cursor()
cur.execute("create table foo(x);")
cur.executemany("insert into foo (x) values(?)", ( [1], [2], [3] ) )

# You can also use it for statements that return data
for row in cur.executemany("select * from foo where x=?", ( [1], [2], [3] ) ):
    print row
```  
  
pysqliteではSELECTを含むSQLでexecutemany()は使用できません。  
http://stackoverflow.com/questions/14142554/sqlite3-python-executemany-select  
  
### コールバック中のエラーが追跡がしやすい  
ユーザ定義関数内でエラーが発生したときなどAPSWは追跡が容易な例外を出力します。  
  
以下にユーザ定義関数内で例外を発生させた場合の違いを確認してみます。  
  
**:pysqliteの例外**  
```py:pysqliteの例外
import sqlite3
def badfunc(t):
    return 1/0

# sqlite3.enable_callback_tracebacks(True)
con = sqlite3.connect(":memory:")
con.create_function("badfunc", 1, badfunc)
cur = con.cursor()
cur.execute("select badfunc(3)")
```  
  
enable_callback_tracebacksがFalse(デフォルト)では以下のようなエラーになります。  
  
```
Traceback (most recent call last):
  File "test_fnc1.py", line 9, in <module>
    cur.execute("select badfunc(3)")
sqlite3.OperationalError: user-defined function raised exception
```  
  
enable_callback_tracebacksがTrueでは以下のようなエラーになります。  
  
```
Traceback (most recent call last):
  File "test_fnc1.py", line 3, in badfunc
    return 1/0
ZeroDivisionError: integer division or modulo by zero
Traceback (most recent call last):
  File "test_fnc1.py", line 9, in <module>
    cur.execute("select badfunc(3)")
sqlite3.OperationalError: user-defined function raised exception
```  
  
enable_callback_tracebacksがFalseの場合は、ユーザー定義関数内の例外が握りつぶされており、仮に、これをTrueにした場合でもTracebackの表示のされ方が直観的なものとは言えないでしょう。  
  
一方、APSWの例外を見てみます。  
  
**:APSWでの例外**  
```py:APSWでの例外
def badfunc(t):
    return 1/0


import apsw
con = apsw.Connection(":memory:")
con.createscalarfunction("badfunc", badfunc, 1)
cur = con.cursor()
cur.execute("select badfunc(3)")
```  
  
APSWでは次のように直観的にわかりやすいTracebackが出力されます。  
  
```
Traceback (most recent call last):
  File "test_fnc2.py", line 9, in <module>
    cur.execute("select badfunc(3)")
  File "c:\apsw\src\connection.c", line 2021, in user-defined-scalar-badfunc
  File "test_fnc2.py", line 2, in badfunc
    return 1/0
ZeroDivisionError: integer division or modulo by zero
```  
  
### APSW Traceによるレポートが出力可能  
APSW Traceは、簡単にあなたのコードを変更せずにSQLの実行をトレースして、要約レポートを提供します。  
  
APSW Traceは下記のソースコード中のtoolsフォルダにあります。  
http://rogerbinns.github.io/apsw/download.html#source-and-binaries  
  
  
 **実行方法**   
  
```
$ python /path/to/apswtrace.py [apswtrace options] yourscript.py [your options]
```  
  
 **実行例**  
「ネストしたトランザクションの利用」で使用したスクリプトのレポートを求めた場合の例を以下に示します。  
  
```
C:\dev\python\apsw>python apswtrace.py --sql --rows --timestamps --thread test_n
est.py
290e5c0 0.002 1734 OPEN: "" win32 READWRITE|CREATE
292aad8 0.009 1734 CURSORFROM: 290e5c0 DB: ""
292aad8 0.010 1734 SQL: create table test(x primary key,y)
290e5c0 0.012 1734 SQL: SAVEPOINT "_apsw-0"
292aad8 0.013 1734 CURSORFROM: 290e5c0 DB: ""
292aad8 0.015 1734 SQL: insert into test values(1,'TEST1')
290e5c0 0.016 1734 SQL: SAVEPOINT "_apsw-1"
292aad8 0.018 1734 CURSORFROM: 290e5c0 DB: ""
292aad8 0.019 1734 SQL: insert into test values(2,'TEST2')
292aad8 0.021 1734 CURSORFROM: 290e5c0 DB: ""
292aad8 0.022 1734 SQL: insert into test values(3,'TEST3')
290e5c0 0.023 1734 SQL: RELEASE SAVEPOINT "_apsw-1"
290e5c0 0.025 1734 SQL: SAVEPOINT "_apsw-1"
292ab10 0.026 1734 CURSORFROM: 290e5c0 DB: ""
292ab10 0.028 1734 SQL: insert into test values(4,'TEST4')
292ab10 0.029 1734 CURSORFROM: 290e5c0 DB: ""
292ab10 0.031 1734 SQL: insert into test values(4,'Error')
290e5c0 0.032 1734 SQL: ROLLBACK TO SAVEPOINT "_apsw-1"
290e5c0 0.034 1734 SQL: RELEASE SAVEPOINT "_apsw-1"
ConstraintError: UNIQUE constraint failed: test.x
rollback 2
290e5c0 0.038 1734 SQL: SAVEPOINT "_apsw-1"
292ab48 0.040 1734 CURSORFROM: 290e5c0 DB: ""
292ab48 0.041 1734 SQL: insert into test values(5,'TEST5')
292ab48 0.043 1734 CURSORFROM: 290e5c0 DB: ""
292ab48 0.044 1734 SQL: insert into test values(6,'TEST6')
290e5c0 0.046 1734 SQL: RELEASE SAVEPOINT "_apsw-1"
290e5c0 0.047 1734 SQL: RELEASE SAVEPOINT "_apsw-0"
292acd0 0.049 1734 CURSORFROM: 290e5c0 DB: ""
292acd0 0.050 1734 SQL: select * from test
292acd0 0.052 1734 ROW: (1, "TEST1")
1 TEST1
292acd0 0.056 1734 ROW: (2, "TEST2")
2 TEST2
292acd0 0.059 1734 ROW: (3, "TEST3")
3 TEST3
292acd0 0.062 1734 ROW: (5, "TEST5")
5 TEST5
292acd0 0.066 1734 ROW: (6, "TEST6")
6 TEST6
APSW TRACE SUMMARY REPORT

Program run time                    0.072 seconds
Total connections                   1
Total cursors                       9
Number of threads used for queries  1
Total queries                       18
Number of distinct queries          14
Number of rows returned             5
Time spent processing queries       0.017 seconds

MOST POPULAR QUERIES

  3 SAVEPOINT "_apsw-1"
  3 RELEASE SAVEPOINT "_apsw-1"
  1 select * from test
  1 insert into test values(6,'TEST6')
  1 insert into test values(5,'TEST5')
  1 insert into test values(4,'TEST4')
  1 insert into test values(4,'Error')
  1 insert into test values(3,'TEST3')
  1 insert into test values(2,'TEST2')
  1 insert into test values(1,'TEST1')
  1 create table test(x primary key,y)
  1 SAVEPOINT "_apsw-0"
  1 ROLLBACK TO SAVEPOINT "_apsw-1"
  1 RELEASE SAVEPOINT "_apsw-0"

LONGEST RUNNING - AGGREGATE

  1  0.017 select * from test
  3  0.000 SAVEPOINT "_apsw-1"
  3  0.000 RELEASE SAVEPOINT "_apsw-1"
  1  0.000 insert into test values(6,'TEST6')
  1  0.000 insert into test values(5,'TEST5')
  1  0.000 insert into test values(4,'TEST4')
  1  0.000 insert into test values(4,'Error')
  1  0.000 insert into test values(3,'TEST3')
  1  0.000 insert into test values(2,'TEST2')
  1  0.000 insert into test values(1,'TEST1')
  1  0.000 create table test(x primary key,y)
  1  0.000 SAVEPOINT "_apsw-0"
  1  0.000 ROLLBACK TO SAVEPOINT "_apsw-1"
  1  0.000 RELEASE SAVEPOINT "_apsw-0"

LONGEST RUNNING - INDIVIDUAL

 0.017 select * from test
 0.000 insert into test values(6,'TEST6')
 0.000 insert into test values(5,'TEST5')
 0.000 insert into test values(4,'TEST4')
 0.000 insert into test values(4,'Error')
 0.000 insert into test values(3,'TEST3')
 0.000 insert into test values(2,'TEST2')
 0.000 insert into test values(1,'TEST1')
 0.000 create table test(x primary key,y)
 0.000 SAVEPOINT "_apsw-1"
 0.000 SAVEPOINT "_apsw-1"
 0.000 SAVEPOINT "_apsw-1"
 0.000 SAVEPOINT "_apsw-0"
 0.000 ROLLBACK TO SAVEPOINT "_apsw-1"
 0.000 RELEASE SAVEPOINT "_apsw-1"

C:\dev\python\apsw>

```  
  
その他、詳細は下記を参考にしてください。  
http://rogerbinns.github.io/apsw/execution.html#apswtrace  
  
  
### pysqlite よりAPSWのほうが早い  
以下のテストではpysqlite よりAPSWのほうが早い結果が出ています。  
http://rogerbinns.github.io/apsw/benchmarking.html  
  
## 参考  
 **APSW 3.8.7.3-r1 documentation » pysqlite differences**   
http://rogerbinns.github.io/apsw/pysqlite.html#pysqlitediffs  
