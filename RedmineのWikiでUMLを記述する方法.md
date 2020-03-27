RedmineのWikiにシーケンス図やユースケースなどのUMLを記述する方法について説明する。  
  
# 前提  
・JAVAが動くこと  
・Redmineが動作すること  
　Apache2でRedmineが動いていたものとする  
  
# 手順  
１.PlantUMLを下記よりダウンロードして任意のフォルダにおく  
http://plantuml.sourceforge.net/download.html  
  
この例では下記に配置したものとする.  
　/share/plantuml.jar  
  
２. ラッパー用のシェルスクリプトを記述する。  
　この例では/usr/bin/plantuml に記述するものとする。  
  
```
# !/bin/bash
/usr/bin/java -Djava.io.tmpdir=/var/tmp -jar /share/plantuml.jar ${@}
```  
  
３.  Redmine用のプラグインを下記からダウンロードする  
https://github.com/cdwertmann/wiki_external_filter  
  
４. 3のファイルをRedmineのpluginsフォルダにコピーする。  
フォルダ名はwiki_external_filterとする。  
ダウンロードした際のデフォルトのフォルダ名はwiki_external_filter_masterになっているので、名前を修正しておく。  
  
保存先の例  
/var/lib/redmine/plugins/wiki_external_filter  
  
５. plugin_assetsフォルダに書き込み権限を与える。  
　/var/lib/redmine/public/plugin_assets/  
  
例：  
chmod go+w /var/lib/redmine/public/plugin_assets/  
  
６. 解凍したディレクトリに存在するconfig/wiki_external_filter.yml を redmineのconfigにコピーする。  
　例：  
　　/var/lib/redmine/config  
  
７. wiki_external_filter.ymlのplantumlにおけるパスを適切に指定する。  
（この例だと修正不要のはず）  
  
  
８. apache2の起動時のlocaleをutf-8とする。  
　これを怠ると、日本語が適切に表示されなくなる。  
  
　/etc/apache2/envvars　の下記を修正  
　export LANG=ja_JP.UTF-8  
　  
　なお、下記の戻り値がUTF-8ならば日本語が使えるようになる。  
　Encoding.find("locale")  
  
９. apache2を再起動  
  
１０.redmineの管理メニューより、キャッシュの保持時間を指定する。  
　　デフォルトは０であるが、この場合は、キャッシュを保持せず画像が絶対に表示されない。  
  
　　管理＞プラグイン＞Wiki External Filter Plugin　の設定  
  
　　「Cache expiration time 」に十分大きな数値を入力  
https://qiita-image-store.s3.amazonaws.com/0/47856/5eae8eeb-0b82-54ae-7467-969e3928628a.png  
  
１１. 下記のような文章をWikiに記述する  
  
```
{{plantuml
ジョニー-> ジャック: 求愛
ジャック-> サラ: 求愛
サラ->ジョニー: 求愛
}}
```  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/4e1dfdb0-17d5-43e4-1bb6-db08f6164987.png  
  
# その他メモ  
・下記のページに一般的なRedmineのプラグインのインストール方法が記載されている。  
http://www.redmine.org/projects/redmine/wiki/Plugins  
  
・正常にプラグインが動作しない場合、ログを出力してデバッグするといい。  
必要な箇所に以下のようなコードを挿入する。  
  
```rb
Rails.logger.info "executing command: #{out['command']}"
```  
  
その結果、以下のようなファイルにログが出力される。  
/var/lib/redmine/log/production.log  
