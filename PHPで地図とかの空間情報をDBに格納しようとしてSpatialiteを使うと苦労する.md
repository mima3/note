## 目的  
この記事ではPHPでSpatialiteを使う方法について解説します。  
  
Spatialiteのビルドなどについては下記を参考にしてください。  
  
 **地図とかの空間情報をSQLiteに格納するSpatiaLiteを使用してみる**   
http://qiita.com/mima_ita/items/64f6c2b8bb47c4b5b391  
  
  
最終目的としては以下のコードが動作することを目標とします。  
  
```php
<?php

class MyDB extends SQLite3
{
    function __construct()
    {
        $this->open(':memory:');
    }
}

$ver = SQLite3::version();
var_dump($ver);


$db = new MyDB();
if (!$db->loadExtension('mod_spatialite.dll')) {
  exit();
}


# reporting some version info
$rs = $db->query('SELECT sqlite_version()');
while ($row = $rs->fetchArray())
{
  print "SQLite version: $row[0]" . PHP_EOL;
}

$rs = $db->query('SELECT spatialite_version()');
while ($row = $rs->fetchArray())
{
  print "SpatiaLite version: $row[0]" . PHP_EOL;
}

$db->exec('SELECT InitSpatialMetaData()');
$db->exec('CREATE TABLE foo (name TEXT)');
$db->exec('SELECT AddGeometryColumn("foo", "Geometry", 0, "POINT", 2)');

# SQLite 3.7.17以降じゃないとRTreeインデックスは作れない
if ($ver->versionNumber < 3007017) {
  print "RTree-Index is Not supported. (< 3.7.17)" . PHP_EOL;
} else {
  $db->exec('SELECT CreateSpatialIndex("foo", "Geometry")');
}
$db->exec('INSERT INTO foo VALUES("test1", GeometryFromText("POINT(140 30)"))');
$db->exec('INSERT INTO foo VALUES("test2", GeometryFromText("POINT(135 33)"))');

$rs = $db->query('SELECT name, AsGeoJson("Geometry") AS geo FROM foo');

while ($row = $rs->fetchArray())
{
  var_dump($row);
}

$db->close();


?>
```  
  
## 前提  
PHP5.5以降でないとloadExtensionは動作しません。  
少なくとも、PHP5.3では基本無理(※）で、PHP5.6ではいけました。  
なので、特に制約がなければPHPの最新を用意してください。  
  
なお、基本無理というのは、機能を落として、なんやかんやすれば、なんとかなりはします。  
  
  
あと、Windowsユーザの人は苦労します。LinuxとかMacのパスの区切りが"/"の環境だと比較的楽です。  
  
  
## php.iniの修正  
loadExtensionを用いて拡張モジュールを使う場合、php.iniを修正して、拡張モジュールを読み込めるようにする必要があります。  
  
Windowsの場合はデフォルトでSQLite3が無効になっている場合もあるのでphp.iniのコメントアウトを外してください。  
  
```ini:php.ini
extension=php_sqlite3.dll
```  
  
次に拡張モジュールを格納しているフォルダを指定します。  
  
```ini:php.ini
[sqlite3]
sqlite3.extension_dir =C:\tool\spatialite\mod_spatialite-4.2.0-win-x86
```  
  
PHPのSQLiteでは、ここで指定したフォルダ以外から拡張モジュールをロードすることはできません。  
  
あとは環境変数のPATHに上記のフォルダを指定すれば、ラッキーガイな環境では動作します。お疲れ様でした。  
  
以下のエラーが出る場合は、ここからが本当の地獄です。  
  
```
PHP Warning:  SQLite3::loadExtension(): 指定されたプロシージャが見つかりません。
```  
  
## 何でエラーになるか  
おそらく、ここでエラーになるのはPHPが使用しているSQLiteのバージョンが古いか、Windowsユーザの人だと思います。  
  
なぜエラーになるかの前にSQLiteの拡張モジュールのエントリーポイントが関係しています。  
  
 **Run-Time Loadable Extensions**   
http://www.sqlite.org/loadext.html  
  
本来のSQLiteのload_extensionはload_extension(X,Y)となっており、２つの引数を取ります。Xがモジュール名でYがエントリーポイントの関数名となります。  
このYは省略可能で、PHPのloadExtension関数の場合、省略して実行されています。  
  
この省略された場合の挙動は次のようになります。  
・もし省略されている場合は　sqlite3_extension_init を使用する  
・これでロードできない場合は、最後の"/" 以降を使用する。この時、接頭語のlibは除去し、アルファベットのみを小文字にし、拡張子は考慮しない。  
具体的には次のようになります。  
  
"/usr/lib/libmathfunc-4.8.so" ⇒ "sqlite3_mathfunc_init".   
"./SpellFixExt.dll" ⇒"sqlite3_spellfixext_init"  
  
  
なお、mod_spatialite.dllのエントリーポイントは「sqlite3_modspatialite_init」となっています。  
  
これについては、Dependency.Walkerで調べるかソースコード読んでください。  
  
![spatialite.png](/image/f6340ec8-6dfe-1865-8ec9-e8701e8a13b9.png)  
http://www.dependencywalker.com/  
  
つまり、PHPからloadExtensionを行った場合にエントリーポイントを「sqlite3_modspatialite_init」に変換できていないのが、このエラーの原因となります。  
  
### 何故、古いPHPではsqlite3_modspatialite_initというエントリーポイントを取得できないか  
PHP5.4以前の場合は話は簡単です。古いSQLiteの仕様にはこの機能がなかったものと推測できます。  
実際、PHP5.4にバンドルされているSQLiteのコードに、該当の処理は存在しません。  
  
https://github.com/php/php-src/blob/PHP-5.4/ext/sqlite3/libsqlite/sqlite3.c  
  
```c
static int sqlite3LoadExtension(
  sqlite3 *db,          /* Load the extension into this database connection */
  const char *zFile,    /* Name of the shared library containing extension */
  const char *zProc,    /* Entry point.  Use "sqlite3_extension_init" if 0 */
  char **pzErrMsg       /* Put error message here if not 0 */
){
// 略
  if( zProc==0 ){
    zProc = "sqlite3_extension_init";
  }

  handle = sqlite3OsDlOpen(pVfs, zFile);
  if( handle==0 ){
    if( pzErrMsg ){
      *pzErrMsg = zErrmsg = sqlite3_malloc(nMsg);
      if( zErrmsg ){
        sqlite3_snprintf(nMsg, zErrmsg, 
            "unable to open shared library [%s]", zFile);
        sqlite3OsDlError(pVfs, nMsg-1, zErrmsg);
      }
    }
    return SQLITE_ERROR;
  }
  xInit = (int(*)(sqlite3*,char**,const sqlite3_api_routines*))
                   sqlite3OsDlSym(pVfs, handle, zProc);
  if( xInit==0 ){
    if( pzErrMsg ){
      *pzErrMsg = zErrmsg = sqlite3_malloc(nMsg);
      if( zErrmsg ){
        sqlite3_snprintf(nMsg, zErrmsg,
            "no entry point [%s] in shared library [%s]", zProc,zFile);
        sqlite3OsDlError(pVfs, nMsg-1, zErrmsg);
      }
      sqlite3OsDlClose(pVfs, handle);
    }
    return SQLITE_ERROR;
  }else if( xInit(db, &zErrmsg, &sqlite3Apis) ){
    if( pzErrMsg ){
      *pzErrMsg = sqlite3_mprintf("error during initialization: %s", zErrmsg);
    }
    sqlite3_free(zErrmsg);
    sqlite3OsDlClose(pVfs, handle);
    return SQLITE_ERROR;
  }

```  
  
見てわかる通り、sqlite3OsDlSymでxInitが取得できなかった場合にエラーとして返しています。  
  
一方、PHP5.5では、xInitが取得できなかった場合に、代替のエントリーポイントを取得する処理が記述されています。  
  
https://raw.githubusercontent.com/php/php-src/PHP-5.5/ext/sqlite3/libsqlite/sqlite3.c  
  
```c
static int sqlite3LoadExtension(
  sqlite3 *db,          /* Load the extension into this database connection */
  const char *zFile,    /* Name of the shared library containing extension */
  const char *zProc,    /* Entry point.  Use "sqlite3_extension_init" if 0 */
  char **pzErrMsg       /* Put error message here if not 0 */
){
// 略
  xInit = (int(*)(sqlite3*,char**,const sqlite3_api_routines*))
                   sqlite3OsDlSym(pVfs, handle, zEntry);

  /* If no entry point was specified and the default legacy
  ** entry point name "sqlite3_extension_init" was not found, then
  ** construct an entry point name "sqlite3_X_init" where the X is
  ** replaced by the lowercase value of every ASCII alphabetic 
  ** character in the filename after the last "/" upto the first ".",
  ** and eliding the first three characters if they are "lib".  
  ** Examples:
  **
  **    /usr/local/lib/libExample5.4.3.so ==>  sqlite3_example_init
  **    C:/lib/mathfuncs.dll              ==>  sqlite3_mathfuncs_init
  */
  if( xInit==0 && zProc==0 ){
    int iFile, iEntry, c;
    int ncFile = sqlite3Strlen30(zFile);
    zAltEntry = sqlite3_malloc(ncFile+30);
    if( zAltEntry==0 ){
      sqlite3OsDlClose(pVfs, handle);
      return SQLITE_NOMEM;
    }
    memcpy(zAltEntry, "sqlite3_", 8);
    for(iFile=ncFile-1; iFile>=0 && zFile[iFile]!='/'; iFile--){}
    iFile++;
    if( sqlite3_strnicmp(zFile+iFile, "lib", 3)==0 ) iFile += 3;
    for(iEntry=8; (c = zFile[iFile])!=0 && c!='.'; iFile++){
      if( sqlite3Isalpha(c) ){
        zAltEntry[iEntry++] = (char)sqlite3UpperToLower[(unsigned)c];
      }
    }
    memcpy(zAltEntry+iEntry, "_init", 6);
    zEntry = zAltEntry;
    xInit = (int(*)(sqlite3*,char**,const sqlite3_api_routines*))
                     sqlite3OsDlSym(pVfs, handle, zEntry);
  }
```  
  
このように古いPHPだとバンドルされているSQLiteのエントリーポイントの取得方法が異なるために、エラーになるのです。  
  
  
### 何故、Windowsではsqlite3_modspatialite_initというエントリーポイントを取得できないか  
  
次にWindowsではどうして新しいPHPでもエントリーポイントを取得できないか説明します。  
PHPはsqlite3.extension_dir で指定したフォルダ名と、loadExtensionで指定したファイル名を組み合わせて、sqlite3LoadExtensionを実行しています。  
  
つまり、sqlite3LoadExtensionに渡されるzFileは次のようになります。  
  
```
C:\tool\spatialite\mod_spatialite-4.2.0-win-x86\mod_spatialite.dll
```  
  
UNIXの場合、パスの区切りが「/」なので適切にファイル名を抽出しますが、Windowsだと「\」なので抽出しません。この場合、エントリーポイントとして期待されるのが以下のようになってしまうのです。  
  
```
ctoolspatialitemod_spatialitewinmod_spatialite
```  
  
## 対応策  
ようするにエントリーポイントを認識させればいいのです。  
残念なことにPHPでは、以下のようなSQLを認めていません。  
  
```sql
SELECT load_extension("mod_spatialite", "sqlite3_modspatialite_init")'
```  
  
https://github.com/php/php-src/blob/PHP-5.5/ext/sqlite3/sqlite3.c  
  
```c:sqlite3.c
PHP_METHOD(sqlite3, loadExtension)
{
	php_sqlite3_db_object *db_obj;
	zval *object = getThis();
	char *extension, *lib_path, *extension_dir, *errtext = NULL;
	char fullpath[MAXPATHLEN];
	int extension_len, extension_dir_len;
	db_obj = (php_sqlite3_db_object *)zend_object_store_get_object(object TSRMLS_CC);

	SQLITE3_CHECK_INITIALIZED(db_obj, db_obj->initialised, SQLite3)

	if (FAILURE == zend_parse_parameters(ZEND_NUM_ARGS() TSRMLS_CC, "s", &extension, &extension_len)) {
		return;
	}

# ifdef ZTS
	if ((strncmp(sapi_module.name, "cgi", 3) != 0) &&
		(strcmp(sapi_module.name, "cli") != 0) &&
		(strncmp(sapi_module.name, "embed", 5) != 0)
	) {		php_sqlite3_error(db_obj, "Not supported in multithreaded Web servers");
		RETURN_FALSE;
	}
# endif

	if (!SQLITE3G(extension_dir)) {
		php_sqlite3_error(db_obj, "SQLite Extension are disabled");
		RETURN_FALSE;
	}

	if (extension_len == 0) {
		php_sqlite3_error(db_obj, "Empty string as an extension");
		RETURN_FALSE;
	}

	extension_dir = SQLITE3G(extension_dir);
	extension_dir_len = strlen(SQLITE3G(extension_dir));

	if (IS_SLASH(extension_dir[extension_dir_len-1])) {
		spprintf(&lib_path, 0, "%s%s", extension_dir, extension);
	} else {
		spprintf(&lib_path, 0, "%s%c%s", extension_dir, DEFAULT_SLASH, extension);
	}

	if (!VCWD_REALPATH(lib_path, fullpath)) {
		php_sqlite3_error(db_obj, "Unable to load extension at '%s'", lib_path);
		efree(lib_path);
		RETURN_FALSE;
	}

	efree(lib_path);

	if (strncmp(fullpath, extension_dir, extension_dir_len) != 0) {
		php_sqlite3_error(db_obj, "Unable to open extensions outside the defined directory");
		RETURN_FALSE;
	}

	/* Extension loading should only be enabled for when we attempt to load */
	sqlite3_enable_load_extension(db_obj->db, 1);
	if (sqlite3_load_extension(db_obj->db, fullpath, 0, &errtext) != SQLITE_OK) {
		php_sqlite3_error(db_obj, "%s", errtext);
		sqlite3_free(errtext);
		sqlite3_enable_load_extension(db_obj->db, 0);
		RETURN_FALSE;
	}
	sqlite3_enable_load_extension(db_obj->db, 0);

	RETURN_TRUE;
}

```  
  
loadExtensionをするまえにsqlite3_enable_load_extensionで拡張モジュールを許可して、処理が終了したら不許可にしていることがわかります。  
つまり、PHPの思想としてはSELECTでload_extensionはさせないというものになります。  
  
これ以外に回避する方法は３つあります。  
  
１つ mod_spatialiteのソースコードでエントリーポイント名を変更する  
２つ mod_spatialite.dllをバイナリエディタで開いて、エントリーポイント名を改ざんする  
３つ sqliteのコードで「\」も考慮するようにする。  
  
以上の３つです。  
  
### mod_spatialiteのソースコードでエントリーポイント名を変更する  
これに関しては未検証です。  
ただ、エントリーポイントを修正するのは楽でも、mod_spatialiteをWindowsでビルドする環境を作るのはしんどいと思います。  
  
  
### mod_spatialite.dllをバイナリエディタで開いて、エントリーポイント名を改ざんする  
  
![spatialite_bin.png](/image/0be5ebef-2ba6-d1e2-d14d-2fdfbbdfd246.png)  
  
![spatialite_bin2.png](/image/f370afef-410c-5205-0eb7-7040ecd89741.png)  
  
sqlite3_modspatialite_initという文字をバイナリエディタで検索して、sqlite3_extension_initに置き換えます。この際、足りないところには00で埋めます。削除したりすると、アドレスが変わるので動作しなくなります。  
  
この方法で対応した場合、RTreeインデックスを使用できなくなることに目をつぶればPHP5.3でもmod_spatialiteを利用できます。  
  
### sqliteのコードで「\」も考慮するようにする。  
そもそも論として、sqlite3.cを以下のように修正すれば「\」でもファイル名のみを抽出します。  
  
```c:sqlite3.c
  if( xInit==0 && zProc==0 ){
    int iFile, iEntry, c;
    int ncFile = sqlite3Strlen30(zFile);
    zAltEntry = sqlite3_malloc(ncFile+30);
    if( zAltEntry==0 ){
      sqlite3OsDlClose(pVfs, handle);
      return SQLITE_NOMEM;
    }
    memcpy(zAltEntry, "sqlite3_", 8);
  	// Windows 対応
    //for(iFile=ncFile-1; iFile>=0 && zFile[iFile]!='/'; iFile--){}
    for(iFile=ncFile-1; iFile>=0 && (zFile[iFile]!='/' && zFile[iFile]!='\\'); iFile--){}
```  
  
あとはPHPのソースコードをWindowsでビルドする方法ですが、以下を参考にすると良いでしょう。  
  
 **PHP拡張モジュールを Windows でビルド**   
http://ngyuki.hatenablog.com/entry/20120625/p1  
  
 **PHP を Windows でビルド**   
http://ngyuki.hatenablog.com/entry/20120701/p1  
  
  
php_sqlite3.dllを作成するには次のようなconfigureでいけました。  
  
```
configure --disable-all --enable-cli --enable-cgi  --with-sqlite3=shared
nmake
```  
  
一応x86でPHP5.6のSQLiteをビルドしたものを以下に置きます。  
http://needtec.sakura.ne.jp/release/php_sqlite3.zip  
  
## まとめ  
このように、SpatialiteをPHPで使うにはかなり苦労が必要です。  
