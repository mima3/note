# Sequelizeとは  
SequelizeはMYSQL,MariaDB,SQLite,Postgresに簡単にアクセスするためのNode.jsのライブラリである。  
  
http://sequelizejs.com  
  
  
以下の機能を有している。  
・オブジェクトとDBの関連を取り持ってくれる。これは１テーブルだけの関係ではなく、複数のテーブルの関連を定義することができる。  
・入力されたデータが適切かどうかのバリデーションチェックを行う。  
・トランザクションのサポートしている。　  
・マイグレーションの機能をサポートしている。これにより、データベースのスキーマの更新が容易になる。  
・1.7.8ではロックの機能は有していないので、自前でSELECT FOR UPDATEなどをしなければならない。しかし、開発中の2.0.0-devにはロックの機能をサポートしている。  
  
# 導入方法  
  
SQLiteを操作する場合  
  
```
npm install --save sequelize
npm install --save sqlite3
```  
  
Postgresを操作する場合  
  
```
npm install --save sequelize
npm install --save pg
```  
  
その他DBについては下記を参照  
http://sequelizejs.com/articles/getting-started  
  
# 実装のサンプル  
## 単純なSQL文の実行  
  
SQLiteに接続し単純なSQLを発行する例を以下に示す  
  
```js
var Sequelize = require('sequelize');
var sequelize = new Sequelize('sample','','',{dialect:'sqlite',storage:'./sample.db'});

sequelize.query('select * from test',null,{raw:true}).success(function(rows) {
  console.log(rows);
});

```  
  
  
## モデルを使った例：  
defineメソッドでオブジェクトの構造を定義できる。  
定義したオブジェクトを経由して、追加、更新、削除が可能だ。  
  
以下の例ではUserモデルを作成し、データを追加後、参照している。  
  
```js
var Sequelize = require('sequelize');
var sequelize = new Sequelize('sample','','',{dialect:'sqlite',storage:'./sample_development.db'});

var User = sequelize.define('User', {
  username: Sequelize.STRING,
  password: Sequelize.STRING
});

sequelize
  .sync({force: true}) // trueだとテーブルを再構築する
  .complete(function(err) {
    if (err) {
      console.log('An error occurred while creating the table:', err);
    } else {
      console.log('It worked');
      // モデル名.buildでインスタンスを新しく作成
      var user = User.build({
        username: 'username',
        password: 'pass'
      });
      // 作成したインスタンスのsaveで保存をする
      user
        .save()
        .complete(function(err) {
          // findまたはfindAllでデータを取得
          User.findAll({where:['id>?',0]}).success(function(result) {
            console.log(result);
          });
        }
      );
    }
  }
);
```  
  
## バリデーションによる値チェックの例：  
バリデーションにより、各フィールドに対する入力制限を指定できる。  
これは、フィールド毎だけでなく、パスワードとユーザ名が同じだったらエラーとするというような柔軟な入力規制も記述できる。  
また、getter,setterを記述することで、必ず小文字に変換するなどという処理が容易に実装可能だ。  
  
その他詳細は下記を参考のこと。  
http://sequelizejs.com/docs/1.7.8/models#validations  
  
  
```js
var Sequelize = require('sequelize');
var sequelize = new Sequelize('sample','','',{dialect:'sqlite',storage:'./sample_development.db'});
var util = require('util');

var User = sequelize.define('User', {
  username: {
    type     : Sequelize.STRING,
    allowNull: false,
    get      : function()  {
      // getするときにデータを補正できる
      return this.getDataValue('username').toString();
    },
    set      : function(v) {
      // setするときにデータを補正できる　この例だと常時小文字で保存
      return this.setDataValue('username', v.toString().toLowerCase());
    },
    validate: {
      // not の場合　@ を含む場合エラーとなる。配列の値はRegExpに渡す値となる。
      // not以外の演算子については下記参照
      // http://sequelizejs.com/docs/latest/models#validations
      not: ['@','i']
    }
  },
  password: Sequelize.STRING
},{
  validate: {
    sameValue: function() {
      // パスワードとユーザ名が両方がひとしければエラーとする
      console.log(this.password);
      console.log(this.username);
      if (this.password == this.username) {
        throw new Error ('sameValue');
      }
    }
  }
});

sequelize
  .sync({force: true}) // trueだとテーブルを再構築する
  .complete(function(err) {
    if (err) {
      console.log('An error occurred while creating the table:', err);
    } else {
      console.log('It worked');
      // bulkCreateを使用すると一度についかできる
      // その他、一気に操作できるものとして、update,destroyなどがある
      // http://sequelizejs.com/docs/latest/instances#bulk
      User.bulkCreate([
        {username: 'ABC1', password:'xxx1'},
        {username: 'abc2', password:'xxx2'},
        {username: 'abc3', password:'xxx3'},
        {username: 'a@c4', password:'xxx4'}, // @が混ざっている
        {username: 'abc5', password:'abc5'}, // パスワードとユーザ名が等しい
      ],null, {validate:true}).success(function() {
        console.log('OK');
      }).error(function(err) {
        // この例だとa@c4とabc5がエラーになる
        console.log(util.inspect(err, true, null));
      });
    }
  }
);
```  
  
## テーブルの関連付けとEager loadingの例：  
外部キーなどで接続してあるテーブルを取得する例を示す。  
http://sequelizejs.com/docs/latest/models#eager-loading  
  
以下のように記述することで、User,Task,TaskGroupの３テーブルをJOINして取得してくれる。  
  
```js
var Sequelize = require('sequelize');
var sequelize = new Sequelize('sample','','',{dialect:'sqlite',storage:'./sample_development.db'});
var async = require('async');

var User = sequelize.define('User', { name: Sequelize.STRING })
  , Task = sequelize.define('Task', { name: Sequelize.STRING })
  , TaskGroup = sequelize.define('TaskGroup', { name: Sequelize.STRING });
 
Task.belongsTo(User);
User.hasMany(Task);

Task.belongsTo(TaskGroup);
TaskGroup.hasMany(Task);

var tasks = [];

// DBの構築
tasks.push(function(next) {
  sequelize.sync({force: true}).done(function() {
    next(null);
  });
});

// ユーザの追加
tasks.push(function(next) {
  User.bulkCreate([
    {name: 'user1'},
    {name: 'user2'}
  ],null, {validate:true}).success(function() {
    next(null);
  }).error(function(err) {
    next(err);
  });
});

// TaskGroupの追加
tasks.push(function(next) {
  TaskGroup.bulkCreate([
    {name: 'gp1'},
    {name: 'gp2'},
    {name: 'gp3'}
  ],null, {validate:true}).success(function() {
    next(null);
  }).error(function(err) {
    next(err);
  });
});

// Taskの追加
tasks.push(function(next) {
  Task.bulkCreate([
    {name: 'task1', UserId:1, TaskGroupId:1},
    {name: 'task2', UserId:1, TaskGroupId:2},
    {name: 'task3', UserId:1, TaskGroupId:3},
    {name: 'task4', UserId:2, TaskGroupId:1}
  ],null, {validate:true}).success(function() {
    next(null);
  }).error(function(err) {
    next(err);
  });
});
async.series(tasks, function(err, results) {
  if (err) {
    console.log(err);
    return;
  }
  User.findAll({ include: [
    {
      model:Task,
      as: 'Tasks',
      attributes: ['id','name'],
      include: [TaskGroup]
    }]}).success(function(users) {
    console.log('User------------------------------------');
    console.log(JSON.stringify(users, null, '  '));
  });
});
```  
  
## Sequelizeでキーを組み合わせてユニーク制約を使う場合  
Sequelizeでキーを組み合わせてユニーク制約を使う場合、次のようにする。  
  
uniqueプロパティはboolean型ならば、その列にユニーク制約を与える。  
文字列の場合は、同じ文字列の列と組み合わせてユニーク制約となる。  
  
もし、別のテーブルと関連づけがあり、自動で生成される列の場合でも、制約をつけたい場合は、明示的に列を指定しなければならない。  
明示的に列を作成した場合は、hasManyなどを行う場合、asでどの列が外部キーであるか指定する必要がある。  
  
```js
module.exports = function(sequelize, DataTypes) {
  var RootPath = sequelize.define('RootPath', {
    path: {
      type: DataTypes.STRING,
      unique: 'projectPathIndex'
    }, 
    ProjectId: {
      type: DataTypes.INTEGER,
      unique: 'projectPathIndex',
      allowNull: false
    }
  }, {
    classMethods: {
      associate: function(models) {
        RootPath.hasMany(models.Project, {as:'ProjectId'});
      }
    }
  });

  return RootPath;
};
```  
  
  
## トランザクションの例：  
Sequelizeではトランザクションを使用することができる。  
http://sequelizejs.com/docs/latest/transactions  
  
以下の例では２つのトランザクションが同時に動いていることが確認できる。  
  
  
```js
var Sequelize = require('sequelize');
var sequelize = new Sequelize('testdb',
                              'postgres',
                              'postgres',
                              {
                                pool: { maxConnections: 10, maxIdleTime: 10000},
                                host:'127.0.0.1',
                                port:5432,
                                dialect:'postgres'
                              });
var async = require('async');


var User = sequelize.define('User', {
  username: Sequelize.STRING,
  password: Sequelize.STRING
});

var tasks = [];

for(var i = 0 ; i < 1; ++i) {
  tasks.push(function(next) {
    doTransaction(10, next);
  });
  tasks.push(function(next) {
    doTransaction(15, next);
  });
}

async.parallel(tasks, function(err, result) {
  User.count().success(function(max) {
    console.log(max);
  }).error(function(error) {
      console.log(error);
  });
});

function doTransaction(cnt, callback) {
  var tasks = [];
  var trn = null;

  tasks.push(function(next) {
    sequelize.transaction(function(t) {
    console.log(t.constructor );
    console.log(t.constructor.LOCK );
      trn = t;
      next(null);
    }).error(function(error) {
      console.log('---------------------------------');
      console.log(error);
      next(error);
    });
  });
  tasks.push(function(next) {
    User.findAll({where: ['id=?',1]}, {transaction:trn}).success(function(data) {
      console.log(data);
      next(null);
    }).error(function(error) {
      console.log(error);
      next(error);
    });
  });
  tasks.push(function(next) {
    User.count({transaction:trn}).success(function(max) {
      console.log(max);
      next(null);
    }).error(function(error) {
      console.log(error);
      next(error);
    });
  });

  for(var i = 0; i < cnt;++i) {
    tasks.push(function(next) {
      User.create({username:'foo'},{transaction:trn}).complete(function(err) {
        next(err);
      });
    });
  }

  tasks.push(function(next) {
    User.count({transaction:trn}).success(function(max) {
      console.log(max);
      next(null);
    }).error(function(error) {
      console.log(error);
      next(error);
    });
  });

  async.series(tasks, function(err, results) {
    console.log(err);
    if (err) {
      callback(err);
      return;
    }
    if(trn) {
      trn.commit().success(function() {
        console.log('OK');
        callback(null);
      }).error(function(error) {
        console.log(error);
        callback(error);
      });
    }
  });
}
```  
  
# sequelizeによるマイグレーションの方法  
sequelize-cliをインストールすることにより、DBのマイグレーションが可能になる。  
この機能により、本来は困難であるはずのDBスキーマーの更新を容易にすることが可能になる。  
  
下記を参考にすること。  
http://sequelizejs.com/docs/latest/migrations  
  
## インストール方法  
  
```
npm install -g --save sequelize-cli
```  
  
以降、sequelizeというコマンドが使用可能になる。  
※使用するDBのライブラリもインストールすること  
  
## 初期化処理  
  
 __コマンド__   
  
```
sequelize init
```  
  
 __説明__   
このコマンドにより、カレントフォルダの初期化を行う。  
configディレクトリとmigrationフォルダを作成する。  
configディレクトリはDBへの接続情報を記述するconfigである。  
test用、development用、production用の３つが作成できる。デフォルトはdevelopmentとなり、これを変更するには--envオプションを使用する。  
migrationディレクトリには、バージョンアップ用、バージョンダウン用の実装を行うためのマイグレーション用のファイルを格納する。  
  
## スケルトンの作成  
  
 __コマンド__   
  
```
sequelize migration:create
```  
  
 __説明__   
migrationフォルダに新規のマイグレーション用のマイグレーションのファイルを作成する。  
作成されるテンプレートは以下のようになる。  
  
```js
module.exports = {
  up: function(migration, DataTypes, done) {
    // add altering commands here, calling 'done' when finished
    done()
  },
  down: function(migration, DataTypes, done) {
    // add reverting commands here, calling 'done' when finished
    done()
  }
}
```  
  
upメソッドにはバージョンアップ時の処理を記述する。  
downメソッドにはバージョンを元に戻すための処理を記述する。  
この際、処理がすべて終わったらdone()を行うこと。  
  
実際のdbに対する操作は、migrationの下記のメソッドを用いて操作を行う  
  
crateTable: テーブルの作成  
dropTable: テーブルの削除  
dropAllTables : 全てのテーブルの削除  
renameTable：　テーブルの名称を変更  
addColumn:列の追加  
removeColumn：列の削除  
changeColumn:列の属性情報を変更する  
renameColumn：列の命名変更  
addIndex:インデックスの追加  
removeIndex: インデックスの削除  
  
  
以下はバージョンアップでテーブルを追加する例になる  
  
```js
module.exports = {
  up: function(migration, DataTypes, done) {
    // add altering commands here, calling 'done' when finished
    migration.createTable(
      'nameOfTheNewTable',
      {
        id: {
          type: DataTypes.INTEGER,
          primaryKey: true,
          autoIncrement: true
        },
        createdAt: {
          type: DataTypes.DATE
        },
        updatedAt: {
          type: DataTypes.DATE
        },
        attr1: DataTypes.STRING,
        attr2: DataTypes.INTEGER,
        attr3: {
          type: DataTypes.BOOLEAN,
          defaultValue: false,
          allowNull: false
        }
      }
    );
    done()
  },
  down: function(migration, DataTypes, done) {
    // add reverting commands here, calling 'done' when finished
    migration.dropTable('nameOfTheNewTable')
    done()
  }
}
```  
  
## マイグレーションの実行  
  
 __コマンド__   
  
```
sequelize db:migrate
```  
  
 __説明__   
実行していないマイグレーションファイルを実行する。  
もし複数マイグレーションのファイルが存在がある場合は、一回のコマンドですべて実行する。  
  
このコマンドを実行した際に、DBにSequelizeMetaテーブルを作成する。  
このテーブルには、このDBにどのマイグレーションファイルが適用されたかを記述するテーブルである。  
  
## マイグレーションの取り消し  
  
 __コマンド__   
  
```
sequelize db:migrate:undo
```  
  
 __説明__   
最後に実行したマイグレーションの処理を取り消す。  
複数回繰り返すことにより、最初の状態まで復帰できる。  
