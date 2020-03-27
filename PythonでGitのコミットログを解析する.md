この記事ではPythonのGitPythonを用いてGITのコミットログを解析する方法について解説する。  
  
前提条件として、下記の通り  
・Gitがインストールされていること。  
・Python2.7系であること。  
　Python3.3への互換性は現時点でないようだ（ゴールに設定されている）  
  
 __開発ソース__   
https://github.com/gitpython-developers/GitPython  
  
 __ドキュメント__   
http://pythonhosted.org/GitPython/0.3.2/  
  
# インストール  
  
以下を実行する。  
  
```
# easy_install GitPython
```  
  
# サンプル  
  
## 指定のリポジトリのコミットの一覧を列挙する  
  
以下のサンプルは指定のリポジトリのコミットのハッシュIDの一覧と、コミット情報を格納するクラス名を出力するスクリプトである。  
  
```py
# -*- coding: utf-8 -*-
from git import *

repo = Repo("/share/testgit/searchTwitter")
for item in repo.iter_commits('master', max_count=100):
  print(item.hexsha)
  print(item.__class__)

```  
  
Repoにはローカルのリポジトリへのパスを入力すること。  
Subversionのような集中管理システムと違い、GITはローカルに構成管理に必要なすべての情報を保持している。この情報はリポジトリの.gitフォルダの中にすべて格納されている。  
  
このコマンドを実行すると、コミットの情報は[git.objects.commit.Commit](http://pythonhosted.org/GitPython/0.3.2/reference.html#module-git.objects.commit "git.objects.commit.Commit")に格納されていることがわかる。  
  
## Commitの主なプロパティ  
  
|名前|説明|  
|:---|:---|  
|author|その作業をもともと行った人|  
|authored_date|Author の日付時刻|  
|author_tz_offset|authorのタイムゾーンのオフセット|  
|committer|その作業を適用した人|  
|committed_date|committerの日付時刻|  
|committer_tz_offset|committerのタイムゾーンのオフセット|  
|message|コミットメッセージ|  
|summary|コミットメッセージの１行目|  
|stats|Diffから作られる統計情報。stats.filesに更新があったファイルの情報が格納されている。|  
|parents|親になる[git.objects.commit.Commit](http://pythonhosted.org/GitPython/0.3.2/reference.html#module-git.objects.commit "git.objects.commit.Commit")の一覧。親がないのが初回コミットになる。これを利用すればコミットの順番を作成できる。|  
|tree|blobを格納するツリー構造のデータ。[Tree](http://pythonhosted.org/GitPython/0.3.2/reference.html#module-git.objects.tree "Tree")クラスで定義されている|  
  
## Treeの主な構造  
ツリー構造を表現するデータ。[Tree](http://pythonhosted.org/GitPython/0.3.2/reference.html#module-git.objects.tree "Tree")クラスで定義されている。  
  
tree.blobsにはツリーに所属するすべてのBlobが格納されているので、以下のように再帰的に呼び出せば、コミットに紐づくBlobをすべて抜き出せる。  
  
```py
def show_tree(tree, indent):
  """
  Treeの情報を出力
  """
  print ('%shexsha :%s' % (indent, tree.hexsha))
  print ('%spath :%s' % (indent, tree.path))
  print ('%sabspath :%s' % (indent, tree.abspath))
  print ('%smode :%s' % (indent, tree.mode))
  for t in tree.trees:
    show_tree(t, indent + '  ')

  print ('%s[blobs]' % indent)
  for b in tree.blobs:
    show_blob(b, indent + '  ')
```  
  
  
Blobは実際のファイルの内容を表すもので、自身のサイズと内容から計算される SHA-1 ハッシュによって名付けられる。  
  
また、コミットのtreeに紐づくBlobは変更があったものだけでなく、すべてのファイルが紐づいている。  
変更がないものは前回のコミットと同じハッシュ値、変更があったものは違うハッシュ値で格納されている。。  
  
Gitは「ディレクトリのスナップショットを保全している」ことがこれからわかると思う。  
  
## 最終的なサンプル  
リポジトリのコミットを抜き出し、各コミットのBlobまで抜き出すサンプルは以下の通りになる。  
  
```py
# -*- coding: utf-8 -*-
from git import *
import time

def show_blob(b, indent):
  """
  Blobの情報を出力
  """
  print ('%s---------------' %(indent))
  print ('%shexsha:%s' % (indent,b.hexsha))
  print ('%smime_type:%s' % (indent,b.mime_type))
  print ('%spath:%s' %(indent,b.path))
  print ('%sabspath:%s' %(indent,b.abspath))

def show_tree(tree, indent):
  """
  Treeの情報を出力
  """
  print ('%shexsha :%s' % (indent, tree.hexsha))
  print ('%spath :%s' % (indent, tree.path))
  print ('%sabspath :%s' % (indent, tree.abspath))
  print ('%smode :%s' % (indent, tree.mode))
  for t in tree.trees:
    show_tree(t, indent + '  ')

  print ('%s[blobs]' % indent)
  for b in tree.blobs:
    show_blob(b, indent + '  ')

def show_commitlog(item):
  """
  Commitの情報を出力
  """
  print ("hexsha %s" %item.hexsha)
  print (item.author)
  print (item.author_tz_offset)
  print (time.strftime("%a, %d %b %Y %H:%M", time.gmtime(item.committed_date)))
  print (item.committer)
  print (item.committer_tz_offset)
  print (item.encoding)
  print (item.message)
  print (item.name_rev)
  print (item.summary)
  print ('[stats]')
  print (item.stats.total)
  print (item.stats.files)
  print ('[parents]')
  for i in item.parents:
    print('  %s' % i.hexsha)

  print '[Tree]'
  show_tree(item.tree, '  ')
    

repo = Repo("/share/testgit/searchTwitter")
for item in repo.iter_commits('master', max_count=100):
  print ('================================')
  show_commitlog(item)

```  
  
# まとめ  
ローカルにCloneしたGitのリポジトリに対してはGitPythonを用いることで、コミットログを簡単に解析できることを説明した。  
  
これを利用することにより、リポジトリに対するコミットの統計情報を作成し、プロジェクト管理の助けに利用することが期待できる。  
  
# 参考  
 __GitPython Documentation__   
https://pythonhosted.org/GitPython/0.3.2/index.html  
  
 __見えないチカラ__   
http://keijinsonyaban.blogspot.jp/2011/05/git.html  
  
 __Git Book__   
http://git-scm.com/book/ja/  
