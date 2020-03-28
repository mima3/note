このドキュメントではRedmineでGitを連携させる方法について記述する。  
  
# 環境  
OS:Debian7  
Redmine version 2.3.1.stable  
git version 1.7.10.4  
  
apt-getでRedmineとgitをインストールしている。  
  
# gitリポジトリの作成してSmartHTTP経由参照できるようにする。  
  
## リポジトリの作成  
Webサーバからアクセスできる権限で、リポジトリを作成する。  
  
```
$mkdir -p /var/git/test.git
$cd /var/git/test.git
$git --bare init
$git update-server-info
$chown -R www-data .
```  
  
## パスワードファイルの作成  
  
```
$htdigest -c /var/git/git.htdigest Git admin
```  
  
## Apacheの設定  
以下のファイルにgitリポジトリの情報を追加して、git-http-backendが動作するようにする。  
  
/etc/apache2/sites-available/default  
  
**/etc/apache2/sites-available/default**  
```text:/etc/apache2/sites-available/default
        SetENV GIT_PROJECT_ROOT /var/git
        SetENV GIT_HTTP_EXPORT_ALL
        ScriptAlias /git "/usr/lib/git-core/git-http-backend"
        <Location /git>
            AuthType Digest
            AuthDigestProvider file
            AuthName "Git"
            AuthUserFile /var/git/git.htdigest
            Require valid-user
            Order allow,deny
            Allow from all
        </Location>
```  
  
## クライアントで確認  
以上の設定でHTTP経由でclone,pushが行えるようになる。  
  
```
$git clone http://debian/git/test
# ローカルレポジトリの操作
$ git push origin master
Username for 'http://debian': admin
Password for 'http://admin@debian':
Counting objects: 5, done.
Writing objects: 100% (3/3), 263 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To http://debian/git/test
   2eb5859..62f3a48  master -> master
```  
  
# Redmineとリポジトリの関連  
## プロジェクトにリポジトリの追加  
任意のプロジェクトで「設定」を選択して「リポジトリ」タブを選ぶ。  
![redminegit1.png](/image/f73c009c-9cd7-30cf-9849-48b6e8601a78.png)  
  
「新しいリポジトリ」を押下すると次の画面が表示される。  
  
![redminegit2.png](/image/c12df7c7-d4fa-e7e6-800c-257b3204f1ed.png)  
  
下記のように設定をする。  
  
|項目|設定|  
|:---|:---|  
|バージョン管理システム|Git|  
|メインリポジトリ|チェックON|  
|識別子|test ... 任意の値|  
|リポジトリのパス|先に設定したGitへのパス<BR>/var/git/test.git|  
|パスのエンコーディング|ブランク<BR>※デフォルトでUTF-8|  
|ファイルとディレクトリの最新コミットを表示する|ON|  
  
ここまで行うと、もしリポジトリにコミットが存在すれば、各プロジェクトのリポジトリ画面には下記のように表示される。  
  
![redminegit3.png](/image/894c9b46-782b-97ed-9cdb-f9e60f9f558a.png)  
  
## チケットとの連携  
「管理」→「設定」のリポジトリタブを選択する。  
  
![redminegit4.png](/image/4ca536ef-fe8f-879f-abf2-fb86a70457a9.png)  
  
ここで以下の設定を行う。  
  
|項目|設定|  
|:---|:---|  
|コミットを自動取得する|チェックON|  
|リポジトリ管理用のWebサービスを有効にする|チェックON|  
|APIキー|のちに使用するので、キーの生成を行いAPIキーを控えておく|  
|参照用キーワード|コミットログにチケットを関連付けるためのキーワード。以下のようにコミットログに記述すると関連づく<BR>refs #12|  
|修正用キーワード|コミット時にステータスを遷移させるためのキーワード。<BR>適用されるステータスを「終了」にしておく|  
  
次に、pushが発生した場合にリポジトリを即更新するようにする。これをおこなわないと、リポジトリ画面を開くまで、チケットとリポジトリが関連づかない。  
  
gitのリポジトリ中のpost-receiveフックを作成する。  
  
**/var/git/test.git/hooks/post-receive**  
```text:/var/git/test.git/hooks/post-receive
# !/bin/sh
wget  http://localhost/redmine/sys/fetch_changesets?key=作成したAPIキー
```  
  
このとき、スクリプトのオーナーがwww-data等のWebサーバーからアクセスできるものになっていること。  
  
  
以降、クライアントからpushを行った場合に、「refs #123」「fixes #123」等のコミットログが存在していれば下記のようにチケットとリポジトリが関連づく  
![redminegit4.png](/image/284865db-bbae-6197-2ac0-c9d556cfcdb1.png)  
  
## チケット番号のないpushを拒否するようにする。  
下記のupdateフックスクリプトを利用する。  
https://github.com/bleis-tift/Git-Hooks  
  
このupdateスクリプトは、push時のコミットで、一行目がチケット番号でないものがあったら、拒否するようになっている。  
  
以下はTortoiseGITでpushが失敗した例である。  
![redminegit3.png](/image/f1e9c739-0960-68be-c296-f0b8ffee6009.png)  
  
  
### コミットログの修正方法  
#### 直前のコミットログの修正方法  
下記のコマンドを入力して、コミットログを修正する。  
  
```
git commit --amend
```  
  
#### 特定のコミットログを修正する  
rebaseを使用する。  
  
2個前のコミットを直す場合は次のようにする。  
  
```
git rebase -i HEAD~2 -i
```  
  
次のような画面が表示される。  
![redminegit3.png](/image/d5b361ed-c136-89f1-eab1-a949c6baf6e4.png)  
  
修正したいコミットのpickをeditに変更する。  
![redminegit3.png](/image/7b5e581c-75d0-7f19-b4e2-4ccdb98ae06d.png)  
  
以下のコマンドを実行してコミットをし直し、rebaseを続ける。  
  
```
$ git commit --amend -m "refs #12"
$ git rebase --continue
```  
  
これで履歴が変わるのでpushを再度実行する。  
  
# 参考  
 __Redmine と Git の連携設定__   
http://easyramble.com/connect-redmine-with-git.html  
  
 __[Git][Deiban] Debian GNU/Linux で Git を Smart HTTP Transport で使う__   
http://d.hatena.ne.jp/next49/20130415/p2  
  
 __Redmine Guide日本語訳 > リポジトリ__   
http://redmine.jp/guide/RedmineRepositories/  
  
 __Redmine とリポジトリの同期設定__   
http://d.hatena.ne.jp/yun_kichi/20100127/1264593665  
