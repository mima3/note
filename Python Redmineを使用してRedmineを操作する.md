# Python Redmine  
  
Python RedmineはRedmineと通信を行うPythonのライブラリである。  
  
次のような特徴を有している。  
  
・RedmineのAPI機能を100%サポート  
　（プロジェクトすら作成できる)  
・Python 2.7、3.4-3.7  
　※2.6,3.3のサポートは2.2.0（2019-01-13）にて削除されました。  
　https://python-redmine.com/changelog.html#id1  
・ORMスタイルのPythonic APIを提供している。  
　（Django ORMに影響を受けています）  
・Apache2.0ライセンス  
  
以下にその詳細が記述してある。  
https://python-redmine.com/  
  
## インストール方法  
pipまたはeasy_installを用いてインストールが可能である。  
  
```
$ pip install python-redmine
```  
  
または  
  
```
$ easy_install python-redmine
```  
  
Redmine側の設定として、「管理」→「設定」画面におけるAPIタブで「REST APIによるWebサービスを有効にする」にチェックをいれておくこと。  
  
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/ea42ef5c-36a8-582f-782e-b342cc41f85c.png  
  
ここでAPIを有効にすると個人設定画面でAPIキーを確認できる。  
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/e7827fdb-1bf2-8c7c-4edb-4e517ed7909e.png  
  
  
## サンプル  
以下にサンプルを記述する。  
このサンプルはWindows上のPython3.7+python redmine2.2.1+Redmine 4.0.3.stableで確認している。  
  
### 認証方法  
ユーザ名とパスワードを指定してredmineに接続できる。  
  
```py
from redminelib import Redmine
redmine = Redmine('http://localhost/redmine', username='admin', password='admin')
```  
  
あるいは次のように、APIキーを用いて接続することも可能だ。  
  
```py
from redminelib import Redmine
redmine = Redmine('http://localhost/redmine', key='e4a413a3b7a3c238102b7393c035bbc5f5eb6409')
```  
  
### チケットの操作  
チケットの操作については下記を参照  
https://python-redmine.com/resources/issue.html  
  
#### チケットの作成  
redmine.issue.new()でチケットのオブジェクトを生成して、そこにプロパティを設定後、保存することでチケットの作成ができる。  
  
```py
import datetime
from redminelib import Redmine
redmine = Redmine('http://192.168.0.200/', key='60076966cebf71506ae3f2391da649235a2b1d46')
issue = redmine.issue.new()
issue.project_id = 'Test1'
issue.subject = 'サブジェクト'
issue.tracker_id = 1     #トラッカー
issue.description = 'チケットの内容をしめす。\n改行もできる。'
issue.status_id = 1      #ステータス
issue.priority_id = 1    #優先度
issue.assigned_to_id = 1 #担当者のID
issue.watcher_user_ids = [1] # ウォッチするユーザのID
issue.parent_issue_id = 12     # 親チケットのID
issue.start_date = datetime.date(2014, 1, 1) #開始日
issue.due_date = datetime.date(2014, 2, 1)   #期日
issue.estimated_hours = 4   # 予想工数
issue.done_ratio = 40
issue.custom_fields = [{'id': 1, 'value': 'foo'}]
issue.uploads = [{'path': 'C:\\dev\\python3\\redmine\\test.txt'}]
issue.custom_fields = [{'id': 1, 'value': 'foo'}]
issue.save()
```  
  
#### 単一チケットの取得  
redmine.issue.get()でチケットIDを指定する。もし存在しない場合、  
redmine.exceptions.ResourceNotFoundError例外が発生する。  
  
```py
import datetime
from redminelib import Redmine
from redminelib.exceptions import ResourceNotFoundError

redmine = Redmine('http://192.168.0.200/', key='60076966cebf71506ae3f2391da649235a2b1d46')
try:
    issue = redmine.issue.get(60)
    print (dir(issue))
    print ('id:%d' % issue.id)
    print ('project:%s' % issue.project.name)
    print ('project_id:%d' % issue.project.id)
    print ('subject:%s' % issue.subject)
    print ('tracker:%s' % issue.tracker.name)
    print ('tracker_id:%d' % issue.tracker.id)
    print ('description:%s' % issue.description)
    print ('status:%s' % issue.status.name)
    print ('status:%d' % issue.status.id)
    print ('author:%s' % issue.author.name)
    print ('author_id:%d' % issue.author.id)
    if hasattr(issue, 'assigned'):
        print ('assigned:%s' % issue.assigned_to.name)
        print ('assigned_id:%d' % issue.assigned_to.id)
    print ('watcher--------')
    for u in issue.watchers:
        print (' %d:%s' % (u.id, u.name))
    print ('作成日:%s' % issue.created_on)
    print ('更新日:%s' % issue.updated_on)
    if hasattr(issue, 'start_date'):
        print ('start_date:%s' % issue.start_date)
    if hasattr(issue, 'due_date'):
        print ('issue_date:%s' % issue.due_date)
    if hasattr(issue, 'issue.estimated_hours'):
        print ('estimated_hours:%d' % issue.estimated_hours)
    print ('作業時間:%d' % issue.spent_hours)
    print ('作業時間の記録----------')
    for t in issue.time_entries:
        print('  ID:%d' % t.id)
        print('  活動:%s' % t.activity)
        print('  コメント:%s' % str(t.comments))
        print('  作成日:%s' % t.created_on)
        print('  時間:%s' %t.hours)
        print('  チケットID:%s' % t.issue)
        print('  プロジェクトID:%s' % t.project)
        print('  日付:%s' % t.spent_on)
        print('  更新日:%s' % t.updated_on)
        print('  user:%d %s' % (t.user.id,t.user.name)) 
    print ('done_ratio:%d' % issue.done_ratio) 
    print ('priority:%s' % issue.priority.name)
    print ('priority_id:%d' % issue.priority.id)
    print ('custom_fields----')
    for c in issue.custom_fields:
        print ('  %d:%s = %s' % (c.id, c.name, c.value))
    print ('attachements---')
    for f in issue.attachments:
        print ('  id:%d' % (f.id))
        print ('  author:%s' % (f.author)) 
        print ('  content_url:%s' % (f.content_url))
        print ('  created_on:%s' % (f.created_on))
        print ('  description:%s' % (f.description))
        print ('  filename:%s' % (f.filename))
        print ('  filesize:%d' % (f.filesize))
        print ('  ---------------')
    print ('changeset---')
    for c in issue.changesets:
        #コミットログがディクショナリ型として格納
        print ('  %s' % c)
    if hasattr(issue, 'parent'):
        print ('parent:%s' % issue.parent)
    print ('children----------')
    for c in issue.children:
        print ('  %s:%s' % (c.id, c.subject))
    print ('relation----------')
    for r in issue.relations:
        print ('  %d:%d->%d(%s)' % (r.id, r.issue_id, r.issue_to_id, r.relation_type)) 
except (ResourceNotFoundError):
    print ('Not found')

```  
  
入力が省略されている項目は、プロパティ自体が存在していない。  
そのため、hasattrで存在チェックをしなければならない。  
  
watchersや担当者、作成者のユーザ情報には必要最小限のものしか入っていない。login名などを取得したい場合は次のようにユーザIDからユーザの詳細を取得すること。  
  
```py
user = redmine.user.get(u.id)
```  
  
#### すべてのチケットを取得  
redmine.issue.all()で取得することができる。  
省略可能なパラメータとして、以下のものがある。  
  
sort (string) ：並び順  
limit (integer) ：取得数の上限  
offset (integer)：取得開始位置  
  
```py
import datetime
from redminelib import Redmine

redmine = Redmine('http://192.168.0.200/', key='60076966cebf71506ae3f2391da649235a2b1d46')
issues = redmine.issue.all(sort='category:desc')
for issue in issues:
  print ('%d:%s' % (issue.id, issue.subject))
```  
  
#### 特定の条件のチケットの取得  
redmine.issue.filterを用いて特定の条件のチケットを抽出できる。  
以下は、担当者が自分のチケットを抽出している。  
  
```py
import datetime
from redminelib import Redmine

redmine = Redmine('http://192.168.0.200/', key='60076966cebf71506ae3f2391da649235a2b1d46')
issues = redmine.issue.filter(assigned_to_id='me')
for issue in issues:
  print ('%d:%s' % (issue.id, issue.subject))
```  
  
query_id を用いれば、登録済みのクエリーで検索も行える。  
  
##### クエリー用のオペレータ  
結局はREST APIを読んでいるにすぎないのでRedmineがサポートしているオペレータが使用できる。  
http://www.redmine.org/projects/redmine/wiki/Rest_Issues  
  
上記のドキュメントでは「!\*」「>」「<」「*」「~」などの例が紹介されている。  
  
なので例えば、誰にもアサインされていないタスクを取得するには下記のような実装になる。  
  
```python
import datetime
from redminelib import Redmine

redmine = Redmine('http://192.168.0.200/', key='60076966cebf71506ae3f2391da649235a2b1d46')
issues = redmine.issue.filter(assigned_to_id='!*')
for issue in issues:
  print ('%d:%s' % (issue.id, issue.subject))
```  
  
また、自分以外の担当者のタスクを取得したければ以下のようになる。  
  
```python
import datetime
from redminelib import Redmine

redmine = Redmine('http://192.168.0.200/', key='60076966cebf71506ae3f2391da649235a2b1d46')
issues = redmine.issue.filter(assigned_to_id='!me')
for issue in issues:
  print ('%d:%s' % (issue.id, issue.subject))
```  
  
否定、範囲指定などのオペレータを使用することで様々な条件付けが可能になることはわかるが、問題はどのようなオペレータがあるかRESTAPIのドキュメントからはわからないことである。  
  
そのため、実際のコードを参照する必要がある。  
  
**redmine/app/models/query.rb**  
https://github.com/redmine/redmine/blob/9746ab7e5be2db5e2d233ee37365cf21ba4b893a/app/models/query.rb#L254  
  
この実装をみれば、どのようなoperators が定義されているか確認できる。  
ただし、全ての項目で全てのオペレータが使用できるわけではない。  
たとえばsubjectでは「\*:すべて」「!\*:なし」「~:含む」は使用できるが「^:～ではじまる」は使用できない。これが使用できるかどうかは下記のフィルタ設定画面でしていできる条件と同じである。  
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/2139e867-2d41-1230-363c-50acb46a55a3.png  
  
  
  
#### チケットの更新  
redmine.issue.getで取得したチケットは変更して保存することが可能である。  
以下の例はチケットのステータスを変更している例である。  
  
```py
# -*- coding: utf-8 -*-
import datetime
from redmine import Redmine

redmine = Redmine('http://localhost/redmine', key='e4a413a3b7a3c238102b7393c035bbc5f5eb6409')
issue = redmine.issue.get(51)
issue.status_id = 2
issue.save()
```  
  
### 作業時間の操作  
作業時間の操作については下記を参照  
https://python-redmine.com/resources/time_entry.html  
  
※Redmine >= 3.4.0あたりから作業分類を入れる必要があるようです。  
  
#### 作業時間の取得  
以下は作業時間を列挙する例である。  
  
```py
import datetime
from redminelib import Redmine

redmine = Redmine('http://192.168.0.200/', key='60076966cebf71506ae3f2391da649235a2b1d46')
time_entries = redmine.time_entry.all()
for t in time_entries:
    print('  ID:%d' % t.id)
    print('  活動:%s' % t.activity)
    print('  コメント:%s' % str(t.comments))
    print('  作成日:%s' % t.created_on)
    print('  時間:%s' %t.hours)
    print('  チケットID:%s' % t.issue)
    print('  プロジェクトID:%s' % t.project)
    print('  日付:%s' % t.spent_on)
    print('  更新日:%s' % t.updated_on)
    print('  user:%d %s' % (t.user.id,t.user.name))
    print('  作業分類 %d %s' % (t.activity.id, t.activity.name))

```  
  
#### 作業時間の記録  
作業時間を記録する例を以下に示す。  
  
```py
import datetime
from redminelib import Redmine

redmine = Redmine('http://192.168.0.200/', key='60076966cebf71506ae3f2391da649235a2b1d46')
time_entry = redmine.time_entry.new()
time_entry.issue_id = 60
time_entry.spent_on = datetime.date(2014, 1, 14)
time_entry.hours = 3
time_entry.activity_id = 4
time_entry.comments = 'hello'
time_entry.activity_id = 8
time_entry.save()
```  
  
## まとめ  
今回は提供されているAPIの一部しか検証していないが、Web画面で行える操作は網羅できていることが確認できた。  
  
Python Redmineを用いることでPythonでRedmineを自由に操作できることが期待できる。  
そのことは、以下のことを実現できると期待できる。  
  
・自動テスト失敗時にプログラムからチケットを発行する。  
・Excelに記述された設計書をWikiに自動コンバート  
・作業時間を集計して、別のシステムに通知する。  
  
以上  
