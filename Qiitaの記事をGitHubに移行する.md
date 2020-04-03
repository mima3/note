# はじめに  
一つのサービスに依存するのは、リスクだと思うのでQiitaの記事をGitHubに移行するスクリプトを書いてみます。  
  
なお、私の記事は以下のようになりました。  
https://github.com/mima3/note  
  
  
# 動作環境  
Python 3.7.4  
Windows10  
  
# 事前準備  
Qiitaのアクセストークンを取得します。  
  
(1)設定画面のアプリケーションタブから「新しいトークンを発行する」を押下します。  
  
![image.png](/image/1027591c-3943-b695-c493-615457408997.png)  
  
(2)読み取り権限を付けて「発行」を押下します  
![image.png](/image/cc957b75-91f7-9f5e-7fd8-2be56c38061f.png)  
  
(3)アクセストークンをメモしておきます  
![image.png](/image/d28c2a4f-19a6-6255-7e30-3d1bbfaf2d42.png)  
  
# 使用方法  
(1)以下のリポジトリからスクリプトを取得する  
https://github.com/mima3/qiita_exporter  
  
(2)下記の形式でスクリプトを実行する。  
  
```
python qiita_to_github.py [userid] [accesstoken] [保存先フォルダ] [GitHubのブロブのルートURL ex.https://github.com/mima3/note/blob/master]
```  
  
(3)保存先フォルダをGitHubに登録します。  
  
# やっていること  
・QiitaAPIを利用してQiita上の指定ユーザが記載した全てのマークダウンを取得します。  
・画像をローカルにダウロードして、リンクを書き換えます。  
・コードブロック以外の行にて、改行コードの前にスペース2ついれて改行を行います。  
・「#タイトル」という記述があったら「# タイトル」に直します。  
・コードブロックのタイトル（例：「```python:test.py」）が表示されないので対応します。  
・自分の記事へのURLを移行先のURLに修正する  
  
  
# 課題  
・タグとかの表現をどうするか。  
・コメントとかの取り扱いをどうするか。  
・既存の記事に移行先を入れる方法とか。  
・タイトルをファイル名としているが、使っている文字によっては動かない気がする。  
・もしかするとWikiの機能を利用した方がいいかも。。。  
・Twitterの埋め込みは旨く移植できなかった。  
![image.png](/image/c95dfc36-1f1c-84d4-b0c6-bb718d0df4f0.png)  
↓  
![image.png](/image/6c0fae2e-d79e-3e84-6260-6cdab2ac6a0b.png)  
・[GitHub Pages](https://help.github.com/en/github/working-with-github-pages)でJekyllが上手く動作しないマークダウンがある。たとえば{%s}とかいうコードが混ざっていると失敗する。  
![image.png](/image/ae6eeab9-db67-c001-4dd9-064fde7f3fe3.png)  
  
![image.png](/image/c7a2502c-e9fd-eb2c-6ae4-5b04b5dd4eae.png)  
  
　  
