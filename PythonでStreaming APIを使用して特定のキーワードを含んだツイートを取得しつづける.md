# 背景  
TwitterAPIには利用制限があり、特定の期間では指定の回数以上使用することができません。  
以下の記事ではsearch/tweets APIを用いてツイートの検索をおこなっていますが、一回のAPIで取得できる上限は100件、15分で180回しか実行できません。  
  
 **Pythonを用いてTwitterの検索を行う**   
http://qiita.com/mima_ita/items/ba59a18440790b12d97e  
  
これは、少ないデータであれば問題ないのですが、「選挙番組中に#総選挙を含むハッシュタグを検索しつづける」という使い方には不向きです。  
  
そこで、Twitterでは、常時データを取得しつづける方法としてStreaming APIを提供しています。  
  
 **The Streaming APIs**   
https://dev.twitter.com/streaming/overview  
  
# Streaming API  
Streaming APIは開発者に少ない遅延でツイッターの情報を取得しつづけることがあります。  
  
Streaming APIには大きく３つの種類があります。  
  
|名前|説明|  
|:---|:---|  
|Public streams|公開されているツイッターのデータを取得できます。<BR>キーワードや場所でフィルタすることのできる[filter](https://dev.twitter.com/streaming/reference/post/statuses/filter)を利用できます|  
|User streams|特定の認証されたユーザのデータとイベントを取得します。|  
|Site streams|複数のユーザーのデータを取得します。現在ベータ版です。アクセスは、ホワイトリストアカウントに制限されています。|  
  
# Pythonのサンプル  
## 目的  
Pythonで特定のキーワードをDBに登録しつづけます。  
  
## 前提のライブラリ  
**python_twitter**   
https://code.google.com/p/python-twitter/  
PythonでTwitterを操作するライブラリです。  
  
 **peewee**   
https://github.com/coleifer/peewee  
sqlite,postgres,mysqlを使用できるORMです。  
  
## サンプルコード  
  
```py
# -*- coding: utf-8 -*-
# easy_install python_twitter
import twitter
import sys
import codecs
import dateutil.parser
import datetime
import time
from peewee import *


db = SqliteDatabase('twitter_stream.sqlite')


class Twitte(Model):
    createAt = DateTimeField(index=True)
    idStr = CharField(index=True)
    contents = CharField()

    class Meta:
        database = db


# easy_install の最新版でGetStreamFilterがなければ下記のコード追加
# https://github.com/bear/python-twitter/blob/master/twitter/api.py
def GetStreamFilter(api,
                    follow=None,
                    track=None,
                    locations=None,
                    delimited=None,
                    stall_warnings=None):
    '''Returns a filtered view of public statuses.

    Args:
      follow:
        A list of user IDs to track. [Optional]
      track:
        A list of expressions to track. [Optional]
      locations:
        A list of Latitude,Longitude pairs (as strings) specifying
        bounding boxes for the tweets' origin. [Optional]
      delimited:
        Specifies a message length. [Optional]
      stall_warnings:
        Set to True to have Twitter deliver stall warnings. [Optional]

    Returns:
      A twitter stream
    '''
    if all((follow is None, track is None, locations is None)):
        raise ValueError({'message': "No filter parameters specified."})
    url = '%s/statuses/filter.json' % api.stream_url
    data = {}
    if follow is not None:
        data['follow'] = ','.join(follow)
    if track is not None:
        data['track'] = ','.join(track)
    if locations is not None:
        data['locations'] = ','.join(locations)
    if delimited is not None:
        data['delimited'] = str(delimited)
    if stall_warnings is not None:
        data['stall_warnings'] = str(stall_warnings)

    json = api._RequestStream(url, 'POST', data=data)
    for line in json.iter_lines():
        if line:
            data = api._ParseAndCheckTwitter(line)
            yield data


def main(argvs, argc):
    if argc != 6:
        print ("Usage #python %s consumer_key consumer_secret access_token_key access_token_secret #tag1,#tag2 " % argvs[0])
        return 1
    consumer_key = argvs[1]
    consumer_secret = argvs[2]
    access_token_key = argvs[3]
    access_token_secret = argvs[4]
    # UNICODE変換する文字コードは対象のターミナルに合わせて
    track = argvs[5].decode('cp932').split(',')

    db.create_tables([Twitte], True)

    api = twitter.Api(base_url="https://api.twitter.com/1.1",
                      consumer_key=consumer_key,
                      consumer_secret=consumer_secret,
                      access_token_key=access_token_key,
                      access_token_secret=access_token_secret)
    for item in GetStreamFilter(api, track=track):
        print '---------------------'
        if 'text' in item:
            print (item['id_str'])
            print (dateutil.parser.parse(item['created_at']))
            print (item['text'])
            print (item['place'])
            row = Twitte(createAt=dateutil.parser.parse(item['created_at']),
                         idStr=item['id_str'],
                         contents=item['text'])
            row.save()
            row = None

if __name__ == '__main__':
    sys.stdout = codecs.getwriter(sys.stdout.encoding)(sys.stdout, errors='backslashreplace')
    argvs = sys.argv
    argc = len(argvs)
    sys.exit(main(argvs, argc))

```  
  
## 使い方  
  
```
python twitter_stream.py consumer_key consumer_secret access_token_key access_token_secret  #総選挙,#衆院選,選挙
```  
  
アクセストークンの情報とキーワードを指定して一考すると、カレントディレクトリにtwitter_stream.sqliteというSQLITEのデータベースを作成します。  
  
## 説明  
・最新のコードではGetStreamFilterメソッドが提供されていますが、Python2.7のeasy_installで取得できるバージョンでは存在しません。ここでは同じコードを実装しています。  
  
・ここではWindowsで動作させているので、cp932でtrack変数を変換していますが、これはターミナルの文字コードに合わせてください。  
  
・大量のデータが流れているときは気になりませんが、最後にツイートされたフィルター対象のデータの取得には数分の遅延があります。  
  
・基本的にデータベースを登録しておいて、時間のかかる処理は別プロセスでやったほうがいいでしょう。  
  
・過去のデータは取れないので、常駐プロセスにする必要があります。  
  
・created_atはUTC時刻なので日本の時間より9時間おそいです。ここで登録された時間に9時間加算した時刻が日本の時刻になります。  
