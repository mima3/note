この記事ではPythonでpython-svnを用いてSubversionのコミットログを解析する方法について解説する。  
  
ここでは2.7でおこなっているが、Python 3.3, 3.2 と2.7のライブラリが提供されている。  
   
http://pysvn.tigris.org/  
  
# インストール  
以下のいずれかの方法でバイナリ―をインストールする  
  
 __Unix系__   
  
```
sudo apt-get install python-svn
```  
  
 __Windows、MACOS__   
下記からダウンロード  
http://pysvn.tigris.org/project_downloads.html  
  
# ログの取得のサンプルと説明  
以下のサンプルは指定のSubversionのレポジトリの履歴を表示するものである。  
  
  
```svnlog.py
import pysvn
import time
import sys
from collections import defaultdict


class SvnRepController:
  def __init__(self,url_or_path,user,passwd):
    self.client = pysvn.Client()
    self.url_or_path = url_or_path
    self.user = user
    self.passwd = passwd
    self.client.callback_get_login = self.get_login

  def get_login(self, realm, username, may_save):
    return True, self.user,self.passwd, False

  def log(self):
    return self.client.log(self.url_or_path, discover_changed_paths=True)

if __name__ == '__main__':
  argvs = sys.argv
  argc = len(argvs)
  if(argc != 4):
    sys.stderr.write( 'Usage:\n python %s url_or_path user pass' % argvs[0] )
    quit()
  url_or_path = argvs[1]
  user = argvs[2]
  passwd = argvs[3]
  client = SvnRepController(url_or_path,user,passwd)

  logs = client.log()
  for log in logs:
    print ("RevNo:%d" % (log.revision.number))
    print ("Author:%s date:%s" % (log.author, time.ctime(log.date)))
    print (log.message)
    for p in log.changed_paths:
      print ("  %s" % dict(p))
```  
  
以下のようにして実行できる。  
  
```
python svnlog.py http://svn.sourceforge.jp/svnroot/simyukkuri/ "user名" "パスワード"
```  
  
手順を説明する。  
  
(1) pysvn.Client()を生成する。  
  
(2)client.callback_get_loginに認証用の関数を指定する。  
　この関数は認証が必要になった場合にコールバックされるものである。  
　戻り値に認証情報を指定する必要がある。  
  
(3)client.log()を行うことで[PySvnLog](http://sourcecodebrowser.com/pysvn/1.6.0/classpysvn_1_1_pysvn_dict_base.html "PySvnLog")の配列が取得できる。  
  
(4)[PySvnLog](http://sourcecodebrowser.com/pysvn/1.6.0/classpysvn_1_1_pysvn_dict_base.html "PySvnLog")は[PysvnDictBase](http://sourcecodebrowser.com/pysvn/1.6.0/classpysvn_1_1_pysvn_dict_base.html "PysvnDictBase")を継承している。PysvnDictBaseはDirectoryと同様に操作できる。つまり次のようにすれば必要なプロパティが取得できる。  
  
```py
  logs = client.log()
  for log in logs:
    dict(log)
```  
  
ここで取得できた主な項目の説明を下記に示す。  
  
 __PySvnLogの内容:__   
  
|名称|説明|  
|:---|:---|  
|revision.number|リビジョン番号|  
|author|作者|  
|date|コミット時刻.数値で表現されている。|  
|message|コミットメッセージ|  
|changed_paths|PysvnLogChangedPathの一覧|  
  
 __PysvnLogChangedPathの内容:__   
  
|名称|説明|  
|:---|:---|  
|action|操作の種別をあらわす<BR>http://svnbook.red-bean.com/en/1.7/svn.ref.svn.c.update.html|  
|path|操作を行ったパス|  
|copyfrom_path|コピー元のパス|  
|copyfrom_revision|コピー元のリビジョン|  
  
  
(5)あとは必要な集計を行う。  
  
# 応用  
以下に応用例を示す  
  
ファイルの更新履歴から同時に更新される頻度の高いファイルのパターンを分析することも可能である。  
以下の例では単純な出現頻度であるファイルを修正した場合に発生する影響度を分析している。  
  
```py
import pysvn
import time
import sys
from collections import defaultdict

class FilePair:
  def __init__(self,path1,path2):
    self.path1 = path1
    self.path2 = path2
    self.count = 0
    self.reliability1 =0
    self.reliability2 =0


class SvnRepController:
  def __init__(self,url_or_path,user,passwd):
    self.client = pysvn.Client()
    self.url_or_path = url_or_path
    self.user = user
    self.passwd = passwd
    self.client.callback_get_login = self.get_login

  def get_login(self, realm, username, may_save):
    return True, self.user,self.passwd, False

  def log(self):
    return self.client.log(self.url_or_path, discover_changed_paths=True)
  
  def getSimpleLogicalCoupling(self):
    files = defaultdict(int)
    ret = defaultdict(FilePair)
    logs = self.log()
    for log in logs:
      for i in range(0,len(log.changed_paths)-1):
        path1 = log.changed_paths[i]
        if path1.action != "M":
          # 変更以外の場合は無視
          continue

        files[path1.path] += 1

        for j in range(i+1,len(log.changed_paths)-1):
          path2 = log.changed_paths[j]
          if path2.action != "M":
            # 変更以外の場合は無視
            continue
          if path1.path == path2.path:
            continue

          key = "%s %s" % (path1.path , path2.path)
          if( ret.has_key(key) == False ):
            key = "%s %s" % (path2.path , path1.path)
            if( ret.has_key(key) == False ):
              ret[key] = FilePair(path1.path,path2.path)
          ret[key].count += 1

    for k,v in ret.items():
      v.reliability1 = float(v.count) / files[v.path1]
      v.reliability2 = float(v.count) / files[v.path2]
      
    return ret



if __name__ == '__main__':
  argvs = sys.argv
  argc = len(argvs)
  if(argc != 6):
    sys.stderr.write( 'Usage:\n python %s url_or_path user pass min_count min_reliability' % argvs[0] )
    quit()
  url_or_path = argvs[1]
  user = argvs[2]
  passwd = argvs[3]
  min_count = argvs[4]
  min_reliablility = float(argvs[5])
  client = SvnRepController(url_or_path,user,passwd)
  list = client.getSimpleLogicalCoupling()
  print '"Path A","Path B","Count","Count/count of A","Count/count of B","reliability"'
  for k,v in sorted(list.items(), key=lambda x:x[1].count, reverse=True):
    if (v.reliability1 > float(min_reliablility) or v.reliability2 > float(min_reliablility)) and v.count > int(min_count):
      print '"%s","%s","%d","%f","%f","%f"' % (v.path1,v.path2,v.count,v.reliability1,v.reliability2,v.reliability1 if v.reliability1>v.reliability2 else v.reliability2)
```  
  
SampleProjectのリポジトリに対してUser:admin,パスワードadminでログインをして、発生頻度が5回以上で、該当ファイルを変更する場合、最低、８０％の確率で更新されているファイルの組み合わせを取得するには以下のようにすればよい。  
  
```
python svnSimpleLogicalCoupling.py http://127.0.0.1/svn/SampleProject admin admin 5 0.8 > out.csv
```  
  
また次のような事ができると思われる。  
・コミットログを解析してどのような修正があるか分析する。  
・RedmineとTracのチケット番号と関連づいている変更履歴を抜き出し、ファイル毎のバグの発生率を求める。  
・人物毎のコミットの頻度  
  
以上、変更履歴を解析することでさまざまな情報が取得できるものと期待できる。  
  
  
