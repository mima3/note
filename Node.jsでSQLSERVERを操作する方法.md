この記事ではNode.jsを用いてSQLSERVERを操作する方法について紹介する  
  
# node-mssql  
node-mssql を使用すれば、Windows以外のOSからSQLSERVERを操作できる。  
https://github.com/patriksimek/node-mssql  
  
以下のnode-sqlserverでもSQLSERVERを操作できるようだが、Windowsでないと動作しない。  
https://github.com/Azure/node-sqlserver  
  
## インストール方法  
  
```
npm install mssql
```  
  
## 使用方法  
### 接続方法  
もっとも単純な接続方法について以下に示す。  
mssql.connectで接続して、不要になったらcloseを行う  
  
```js
var mssql = require('mssql');
var config = {
  user: 'sa',
  password: 'XXXX',
  server: 'ホスト名\\SQLEXPRESS', // You can use 'localhost\\instance' to connect to named instance
  database: 'Sample001',
  stream: true, // You can enable streaming globally

  options: {
    encrypt: true // Use this if you're on Windows Azure
  }
}
mssql.connect(config, function(err) {
  console.log(err);
  mssql.close();
});
```  
  
### SQLの発行方法  
connectが完了したらRequestオブジェクトを生成して発行する  
SQLSERVERの場合、複数のレコードセットが返ってくる場合があるので、recordsetイベントでそれを検知すること。  
  
```js
    var request = new mssql.Request(); // or: var request = connection.request();
    request.query('select * from T01Prefecture');
    request.on('recordset', function(columns) {
       // レコードセットを取得するたびに呼び出される
       console.log(columns);
    });
    request.on('row', function(row) {
       // 行を取得するたびに呼ばれる
       console.log(row);
    });

    request.on('error', function(err) {
       // エラーが発生するたびによばれる
       console.log(err);
    });
   
    request.on('done', function(returnValue) {
        // 常時最後によばれる
        console.log(returnValue);
    })
```  
  
### PreparedStatement   
PreparedStatement を使用するにはinputメソッドでキーになる文字列と値を関連付ける。  
VARCHARを使用する場合は、mssql.NVarCharを設定しておかないと文字化けが発生するので注意。  
  
```js
  var request = new mssql.Request();
  request.input('id', mssql.Int, 102);
  request.input('name', mssql.NVarChar, 'ロール'); // VARCHARだろうがNVarCharにしとかないと化ける
  request.query('INSERT INTO t01prefecture(PREF_CD,PREF_NAME) VALUES(@id,  @name)');
```  
  
### トランザクション  
mssql.Transaction()を用いてトランザクションを作成できる。  
Request時にそのトランザクションを引数とする。  
クエリー実行後、commitまたはrollbackを行えばよい。  
  
```js
    var tran = new mssql.Transaction();
    tran.begin(function(err) {
    var request = new mssql.Request(tran); // or: var request = connection.request();
      request.query('select * from T01Prefecture');
      request.on('done', function(returnValue) {
          // 常時最後によばれる
          console.log(returnValue);
          tran.commit(function(err, ret) { // or rollback
             // TODO
             console.log('Commit');
          });
      })
    });

```  
  
### サンプルコード  
[Pythonで色々なデータベースを操作する](http://qiita.com/mima_ita/items/9a5ab3b45c7575776b06 "Pythonで色々なデータベースを操作する")と同様のサンプルを動かす  
  
  
```js
var mssql = require('mssql');
var async = require('async');
var util = require('util');

var config = {
  user: 'sa',
  password: 'sa',
  server: 'hostname\\SQLEXPRESS', // You can use 'localhost\\instance' to connect to named instance
  database: 'Sample001',
  stream: true, // You can enable streaming globally

  options: {
    encrypt: true // Use this if you're on Windows Azure
  }
}
var tasks = [];

// 接続
tasks.push(function(next) {
  mssql.connect(config, function(err) {
    next(err);
  });
});

// TESTデータ削除
tasks.push(function(next) {
  var request = new mssql.Request();
  request.input('from', mssql.Int, 100);
  requestSql(request, 'DELETE FROM T01Prefecture WHERE PREF_CD >= @from', function(errors, ret) {
    console.log(util.inspect(ret,false,null));
    next(errors);
  });
});

// PREPARESTATEMENTを用いたSELECT
tasks.push(function(next) {
  var request = new mssql.Request();
  request.input('from', mssql.Int, 45);
  request.input('to', mssql.Int, 999);
  requestSql(request, 'SELECT * FROM T01Prefecture WHERE PREF_CD BETWEEN @from AND @to', function(errors, ret) {
    console.log(util.inspect(ret,false,null));
    next(errors);
  });
});

////////////////////////////////////////
// コミットの試験
////////////////////////////////////////
// トランザクション開始
tasks.push(function(next) {
  var transaction = new mssql.Transaction();
  transaction.begin(function(err) {
    next(err, transaction);
  });
});

tasks.push(function(transaction, next) {
  var request = new mssql.Request(transaction);
  request.input('id', mssql.Int, 100);
  request.input('name', mssql.NVarChar, 'モテモテ国'); // VARCHARだろうがNVarCharにしとかないと化ける
  requestSql(request, 'INSERT INTO t01prefecture(PREF_CD,PREF_NAME) VALUES(@id,  @name) ', function(errors, ret) {
    next(errors, transaction);
  });
});

tasks.push(function(transaction, next) {
  var request = new mssql.Request(transaction);
  request.input('id', mssql.Int, 101);
  request.input('name', mssql.NVarChar, '野望の国'); // VARCHARだろうがNVarCharにしとかないと化ける
  requestSql(request, 'INSERT INTO t01prefecture(PREF_CD,PREF_NAME) VALUES(@id,  @name) ', function(errors, ret) {
    next(errors, transaction);
  });
});

tasks.push(function(transaction, next) {
  var request = new mssql.Request(transaction);
  request.input('from', mssql.Int, 45);
  request.input('to', mssql.Int, 999);
  requestSql(request, 'SELECT * FROM T01Prefecture WHERE PREF_CD BETWEEN @from AND @to', function(errors, ret) {
    console.log('コミット前---------------');
    console.log(util.inspect(ret,false,null));
    next(errors, transaction);
  });
});

tasks.push(function(transaction, next) {
  transaction.commit(function(err, ret) {
     next(err);
  });
});

tasks.push(function(next) {
  var request = new mssql.Request();
  request.input('from', mssql.Int, 45);
  request.input('to', mssql.Int, 999);
  requestSql(request, 'SELECT * FROM T01Prefecture WHERE PREF_CD BETWEEN @from AND @to', function(errors, ret) {
    console.log('コミット後---------------');
    console.log(util.inspect(ret,false,null));
    next(errors);
  });
});

////////////////////////////////////////
// ロールバックの試験
////////////////////////////////////////
tasks.push(function(next) {
  var transaction = new mssql.Transaction();
  transaction.begin(function(err) {
    next(err, transaction);
  });
});

tasks.push(function(transaction, next) {
  var request = new mssql.Request(transaction);
  request.input('id', mssql.Int, 102);
  request.input('name', mssql.NVarChar, 'ロール'); // VARCHARだろうがNVarCharにしとかないと化ける
  requestSql(request, 'INSERT INTO t01prefecture(PREF_CD,PREF_NAME) VALUES(@id,  @name) ', function(errors, ret) {
    next(errors, transaction);
  });
});

tasks.push(function(transaction, next) {
  var request = new mssql.Request(transaction);
  request.input('from', mssql.Int, 45);
  request.input('to', mssql.Int, 999);
  requestSql(request, 'SELECT * FROM T01Prefecture WHERE PREF_CD BETWEEN @from AND @to', function(errors, ret) {
    console.log('ロールバック前---------------');
    console.log(util.inspect(ret,false,null));
    next(errors, transaction);
  });
});

tasks.push(function(transaction, next) {
  transaction.rollback(function(err, ret) {
     next(err);
  });
});

tasks.push(function(next) {
  var request = new mssql.Request();
  request.input('from', mssql.Int, 45);
  request.input('to', mssql.Int, 999);
  requestSql(request, 'SELECT * FROM T01Prefecture WHERE PREF_CD BETWEEN @from AND @to', function(errors, ret) {
    console.log('ロールバック後---------------');
    console.log(util.inspect(ret,false,null));
    next(errors);
  });
});

// ストアドプロシージャの試験
tasks.push(function(next) {
  console.log('単一のレコードセットを返すストアドプロシージャの試験');
  var request = new mssql.Request();
  request.input('from', mssql.Int, 1);
  request.input('to', mssql.Int, 10);
  requestSql(request, 'exec test_sp @from , @to', function(errors, ret) {
    console.log(util.inspect(ret,false,null));
    next(errors);
  });
});

// 複数のレコードセットを返すストアドプロシージャの試験
tasks.push(function(next) {
  console.log('複数のレコードセットを返すストアドプロシージャの試験');
  var request = new mssql.Request();
  request.input('from', mssql.Int, 1);
  request.input('to', mssql.Int, 10);
  requestSql(request, 'exec test_sp2 @from , @to', function(errors, ret) {
    console.log(util.inspect(ret,false,null));
    next(errors);
  });
});


async.waterfall(tasks, function(err) {
  if(err) {
    console.log(err);
    process.exit();
  }
  mssql.close();
});

function requestSql(request, sql, callback) 
{
  var errors = [];
  var result = [];
  var records = [];
  request.query(sql);
  request.on('recordset', function(columns) {
    // Emitted once for each recordset in a query
    //console.log(columns);
    var rec = {
      columns:columns,
      records: []
    };
    result.push(rec);
  });

  request.on('row', function(row) {
    // Emitted for each row in a recordset
    result[result.length - 1].records.push(row);
  });

  request.on('error', function(err) {
    // May be emitted multiple times
    errors.push(err);
  });

  request.on('done', function(returnValue) {
    console.log(returnValue);
    // Always emitted as the last one
    if (errors.length == 0) {
      callback(null, result);
    } else {
      callback(errors, result);
    }
  });
}


```  
