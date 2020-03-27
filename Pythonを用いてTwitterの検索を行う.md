# 目的  
Pythonを用いてTwitterの検索を行うスクリプトを記述する。  
最終的な成果物については下記からダウンロードできる。  
  
https://github.com/mima3/searchTwitter  
  
# 準備  
(1)Python2.7がインストール済みであること。  
(2)python_twitterをインストールする  
  
```
easy_install python_twitter
```  
  
(3)下記のページよりTwitter用のAPIを取得する  
https://dev.twitter.com/  
  
詳しい取得方法は下記のページを参考にする。  
http://support.dreamone.co.jp/Pandora/dp.do?jumpTo=DreamX&variables%28LPID%29=162  
  
# 使用するTwitterのAPI  
## application/rate_limit_status.json  
https://dev.twitter.com/docs/api/1.1/get/application/rate_limit_status  
各APIの制限を取得する。  
これにより、各APIが後何回使用できるか、そして、リセット時間がいつか調べることができる。  
  
## search/tweets.json  
https://dev.twitter.com/docs/api/1.1/get/search/tweets  
検索用のAPI。  
  
一回のAPIで取得できる上限は100件となる。  
APIを経由した検索は公式ページからの検索と結果が異なる場合がある。  
公式では７日前のツイートでも検索されるが、APIの場合、過去のツイートは検索対象とならない。  
  
また、result_typeにより検索で取得されるツイートがことなる。  
recentは時系列で取得する  
popularは人気のあるツイートを取得する  
mixedは上記をまぜたものである。  
  
  
### 検索文字  
検索APIで指定できる文字は「高度な検索」で使用する検索文字を使用することができる。  
https://twitter.com/search-advanced  
  
#### AND,ORを用いた検索例  
  
```
(えーりん OR 永琳) AND (BBA OR 婆 OR ババア)
```  
  
「えーりん」または「永琳」と「BBA」または「婆」または「ババア」とつぶやかれているものを検索する。  
  
#### 特定のユーザがつぶやいたものを検索する例  
「from:」のあとにユーザー名を入力する。  
  
```
from:mima_ita
```  
  
fromを使用した検索はAPIでは100件が上限だと思われる。  
  
#### 特定の場所でつぶやいたものを検索する例  
「geocode:」の後に座標と範囲を指定する。  
次の例は東京タワー半径500mのつぶやきを取得したものである。  
  
```
geocode:35.65858,139.745433,0.5km
```  
  
geocodeを使用した検索はAPIでは100件が上限だと思われる。  
  
# 実装例  
ここでは指定の検索文字を検索する処理の実装例を示す。  
本来は１回の検索APIで100件しかとれないので、これを改造し制限いっぱい取得できるようにする。  
  
  
まず１回目の検索APIを"result_type=recent"で検索して時系列で取得する。この時取得されるのは最新から過去100件のみである。  
  
二回目の検索では、1回目の検索で取得したもっとも古いツイートより昔のものを取得するようにする。これを行うには"max_id = 前回の最小id-1"を指定すればよい。  
  
すべての１件も取得できなくなるまでこれを繰り返すか、rate_limit_statusで取得したAPIの制限を超えるまで繰り返せばよい。  
  
この簡単のサンプルを以下に示す。  
  
```py
# !/usr/bin/python
# -*- coding: utf-8 -*-
# python_twitter 1.1
import twitter
from twitter import Api
import sys
import time
reload(sys)
sys.setdefaultencoding('utf-8')
from collections import defaultdict



maxcount=1000
maxid =0
terms=["八意永琳","永琳","えーりん"]
search_str=" OR ".join(terms)

api = Api(base_url="https://api.twitter.com/1.1",
                  consumer_key='XXXXX',
                  consumer_secret='XXXXX',
                  access_token_key='XXXXX',
                  access_token_secret='XXXXX')
rate = api.GetRateLimitStatus()
print "Limit %d / %d" % (rate['resources']['search']['/search/tweets']['remaining'],rate['resources']['search']['/search/tweets']['limit'])
tm = time.localtime(rate['resources']['search']['/search/tweets']['reset'])
print "Reset Time  %d:%d" % (tm.tm_hour , tm.tm_min)
print "-----------------------------------------\n"
found = api.GetSearch(term=search_str,count=100,result_type='recent')
i = 0
while True:
  for f in found:
    if maxid > f.id or maxid == 0:
      maxid = f.id
    print f.text
    i = i + 1
  if len(found) == 0:
    break
  if maxcount <= i:
    break
  print maxid
  found = api.GetSearch(term=search_str,count=100,result_type='recent',max_id=maxid-1)

print "-----------------------------------------\n"
rate = api.GetRateLimitStatus()
print "Limit %d / %d" % (rate['resources']['search']['/search/tweets']['remaining'],rate['resources']['search']['/search/tweets']['limit'])
tm = time.localtime(rate['resources']['search']['/search/tweets']['reset'])
print "Reset Time  %d:%d" % (tm.tm_hour , tm.tm_min)
```  
  
  
  
  
  
### 発展系  
上記を発展させたスクリプトを下記からダウンロードできる。  
https://github.com/mima3/searchTwitter  
  
上記のスクリプトは検索結果をSQLITEに保存している。  
このスクリプトはAPIの呼び出し制限ギリギリまで過去のツイートを検索する。  
つぎにスクリプトを実行した場合、以下のようになる。  
  
 __すべての検索可能な過去のツイートを検索済みの場合__   
DBに登録されているツイートより新しいツイートを検索する。  
  
 __過去のツイートが残っている場合__   
前回取得したツイートより古いツイートを検索する。  
  
このスクリプトにより、大量の検索結果を容易に取得することができる。  
  
  
  
