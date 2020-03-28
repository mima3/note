# 目的  
Pythonを用いてfacebookを操作してみる  
  
 **デモページ**   
https://needtec.sakura.ne.jp/check_facebook/  
  
 **Github**   
https://github.com/mima3/check_facebook  
  
  
# 使用ライブラリ  
## facebook-sdk  
facebook-sdkはPythonでFacebook Graph APIを操作するAPI。  
  
https://github.com/pythonforfacebook/facebook-sdk  
  
```
easy_install facebook-sdk
```  
  
## bottle  
PythonのWebフレームワーク。1ファイルのみで構成されているのでインストールが楽。  
  
http://bottlepy.org/docs/dev/index.html  
  
## beaker  
Pythonでセッション管理を行うためのライブラリ  
  
https://beaker.readthedocs.org/en/latest/#  
  
```
easy_install Beaker
```  
  
# Facebookの操作の方法  
## アプリケーションの登録方法  
1.facebookの開発者登録を行う。  
  
 **開発者登録の手順**   
http://fb.dev-plus.jp/what-devplus/dev_register/  
  
2.開発者ページの[Apps]->[Add a New App]を実行する。  
https://developers.facebook.com/  
  
![facebook1.png](/image/dc70636f-7def-4340-467d-2a5352882ab3.png)  
  
3.アプリケーションの種類で[ウェブサイト]を選択する  
![facebook2.png](/image/a165a05f-b75d-4433-4d0d-8e74f3a268c6.png)  
  
4.アプリケーション名を入力して、[Create New Facebook APP ID]を選択する  
  
![facebook3.png](/image/4e9816e5-4cac-36bd-a937-5939bed0f021.png)  
  
5.カテゴリーを選択して[Create APP ID]を入力する。  
![facebook4.png](/image/e663993f-e47f-9458-7c01-d8667bf5b014.png)  
  
6.作成されたページを下にスクロールすると「サイトURL」を入力できるので、そこを入力して「次へ」を押下する  
  
![facebook5.png](/image/797cd78a-88ec-ce67-ce7d-068d7110ddf4.png)  
  
![facebook6.png](/image/1301ac74-38e2-876e-4a0e-24c21f64e59a.png)  
  
FacebookAPIで指定するredirect_urlは、このサイとで指定したドメイン名でなければならない。IPアドレスの指定はできないようなので、ローカルで動かす場合はlocalhostとしておく。  
  
7.ページをリロードするとメニューから作成したアプリケーションを選択できるようになる。  
![facebook8.png](/image/b0afc373-22ee-7d25-8333-1e2b013a67db.png)  
  
8.アプリを選択すると、「App ID」と「App Secret」を表示できる。この値を使って認証を行いアクセストークンを取得できる。  
  
![facebook9.png](/image/b1ec2936-d896-1364-574d-752715c3bc08.png)  
  
## アクセストークン取得する  
以下にアクセストークン取得する方法を示す。  
  
1.oauthに接続してcodeを取得する  
  
 **エンドポイント：**   
https://www.facebook.com/dialog/oauth  
  
 **パラメータ：**   
client_id: facebookアプリのAppID  
redirect_url: 認証後のリダイレクトURL。設定したドメインでないとエラーになる。  
scope:「,」区切りで権限を指定する。  
https://developers.facebook.com/docs/facebook-login/permissions/v2.2?locale=ja_JP  
  
 **例：**   
https://www.facebook.com/dialog/oauth?client_id=XXXXX&redirect_uri=http%3A%2F%2Flocalhost%2Fcheck_facebook&scope=read_stream  
  
OKした場合のリダイレクトURL  
  
```
https://localhost/check_facebook/index.cgi/?code=XXXXX#_=_
```  
  
キャンセルした場合のリダイレクトURL  
  
```
https://localhost/check_facebook/?error=access_denied&error_code=200&error_description=Permissions+error&error_reason=user_denied#_=_
```  
  
2.access_tokenに接続してaccess_tokenを取得する  
  
 **エンドポイント：**   
https://graph.facebook.com/oauth/access_token  
  
 **パラメータ：**   
client_id: facebookアプリのAppID  
client_secret: facebookアプリのSecret  
redirect_url: 認証後のリダイレクトURL。設定したドメインでないとエラーになる。  
code: oauthで取得したCode  
  
 **例：**   
https://graph.facebook.com/oauth/access_token?client_id=facebookアプリのAppID&client_secret=facebookアプリのSecret&redirect_uri=http%3A%2F%2Flocalhost%2Fcheck_facebook&code=oauthで取得したCode  
  
  
エラーの場合:  
  
```
{
   "error": {
      "message": "Error validating application. Invalid application ID.",
      "type": "OAuthException",
      "code": 101
   }
}
```  
  
access_tokenを取得できた場合：  
  
```
access_token=XXX&expires=5183979
```  
  
ここで取得したaccess_tokenを利用してGraphAPIにアクセス  
  
## PythonでBottle+Beakerでセッションを使用する方法  
以下にbottleをCGIで動作した場合のサンプルを以下に示す。  
  
**:index.cgi**  
```py:index.cgi
from bottle import run
from application import app
from beaker.middleware import SessionMiddleware

session_opts = {
    'session.type': 'file',
    'session.data_dir': './session',
    'session.cookie_expires': True,
    'session.auto': True
}
appSession = SessionMiddleware(app, session_opts)
run(appSession, server='cgi')

```  
  
**:application.py**  
```py:application.py
from bottle import get, post, template, Bottle, response, request, redirect
import os

app = Bottle()

@app.get('/')
def index():
    session = request.environ.get('beaker.session')
    session['counter'] = session.get('counter', 0) + 1
    session.save()
    return template('<b>Hello {{name}}</b>!', name=session['counter'])

```  
  
ページにアクセスするたびに、session.data_dirのファイルが更新される。  
Beakerでは、作成されたファイルの削除は行われないので、cronなどで定期的に削除すること。  
  
```
find /hoge/session -type f -mmin +60 -exec rm {} \;
```  
  
## facebook-sdkを使用した操作例  
  
```py
# -*- coding: utf-8 -*-
import facebook
graph = facebook.GraphAPI('取得したAPI')
profile = graph.get_object('facebookpageのIDか名前')
print profile
posts = graph.get_connections(profile['id'], 'posts')
print posts
```  
  
# まとめ  
facebook-sdk、bottle、Beakerを利用することで、PythonでもfacebookAPIを用いたアプリケーションを作成できる。  
