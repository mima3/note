# 目的  
ニコニコカレンダーとはチームのチームのモチベーションやムードを可視化したものです。  
  
http://www.geocities.jp/nikonikocalendar/index_ja.html  
  
この記事ではTracのプラグインとしてニコニコカレンダーを使用できるようにします。  
![b0232065_12251021.png](/image/1a99f68e-5cda-040d-5947-39ca15b34271.png)  
  
このプラグインはBrett Smith氏がTrac0.10系で作成したプラグインをTrac0.12系へ移植して若干の修正を行ったものです。  
  
Niko-Niko Calendar  
http://trac-hacks.org/wiki/NikoNikoPlugin  
  
# 導入方法  
## 前提  
Trac-Lightningの3.2.0により、Trac0.12がインストールされているものとします。  
http://sourceforge.jp/projects/traclight/  
  
## インストール方法  
(1)下記よりダウンロードを行います。  
https://github.com/mima3/nikonikoplugin  
  
(2)Zipを解凍すると「0.10」と「0.12」が存在します。  
今回はTrac0.12が対象なので「0.12」フォルダのsetup.pyを使用します。  
  
(3)TracLightningをインストールしたときに作成されたスタートメニューからコマンドプロンプトを起動してください。  
  
![b0232065_12341098.png](/image/e3936672-ac4f-edfb-68dc-366c5cbebe0d.png)  
  
このメニューから起動された場合、Tracで使用している環境へのパスが通っています。  
Pythonを複数いれている方は必ず、ここから実行しましょう。  
  
(4)2で解凍したフォルダにカレントディレクトリを移動させて、下記のコマンドを実行して、インストールを行います。  
  
```
python setup.py install
```  
  
(5)Apachを再起動して、Tracの管理者権限でログインします。  
「一般設定」の「プラグイン」を選択して「nikoniko」を探してください。  
![b0232065_1240533.png](/image/99ef586e-98a2-0391-f034-581524bd13d3.png)  
  
 nikonikoプラグインをみつけたらNikoNikoComponentにチェックをつけて変更を適用してください。  
  
![b0232065_12422987.png](/image/3096161d-c3fd-6bc9-3445-3c64b2427cd6.png)  
  
適用後、ニコニコカレンダーのページに移動するとエラーがでる可能性があります。  
これはニコニコカレンダーで使用しているテーブルが作成されていないために発生します。  
  
(6)テーブルを作成するために、一旦、Apachを停止してください。  
その後、３で使用したコマンドプロンプトを起動してTrac-Adminを用いてデータベースのアップグレードをおこないます。  
下記のコマンドを入力してください。  
  
```
Trac-Admin "プロジェクトのFullPath" upgrade 
```  
  
TracLightningを既定の値でインストールした場合、Sampleプロジェクトのフルパスは「C:\TracLight\projects\trac\SampleProject」になっていると思います。  
  
(7)Apachを再起動して管理者権限でログインします。  
　管理メニューの権限で「NIKONIKO_CHANGE」と「NIKONIKO＿VIEW」を任意のユーザーに付与してください。  
  
![b0232065_12505856.png](/image/76cc701d-de2e-21af-e4b1-19288c25b90b.png)  
  
(8)するとメニューの右はじに「二コカレ」というメニューが作成されます。  
あとは今日のきもちにあわせて「やる夫」を選んで必要なら、なんかコメントをのこしてください。  
  
## オリジナルとの違い  
・Trac0.10からTrac0.11に移行するさいにテンプレートエンジンがGenshiに変わりました。そのため、テンプレートの周りを修正しました。  
  
・本来ニコニコカレンダーは３つから選択しますが、つい５つにしてしまいました。  
  
・コメントを残せるようにしました。これはマウスでオーバーすることで表示されます。愚痴をかくもよし、笑いをとるもよし。  
  
# TracPluginをつくる場合のメモ  
以下は、TracPluginを改造した時にとったメモです。  
  
・致命的なエラーはTracがログとして出力する。これは、プロジェクトのlogフォルダにtrac.logという名前で作成される。  
ログの出力レベルはtrac.iniで指定できる。  
  
・NikoNikoComponentなどのプラグイン中でログを任意のタイミングで出力できる。  
  
```
from trac.log import logger_factory

# class NikoNikoComponent(Component)中で・・・
self.log.info("days %s %s", locale.getlocale(),days[0])
```  
  
・プラグインのページのURLをテンプレートから利用するのには癖がある  
まず、web_ui.pyで次のようなメソッドを用意しておく  
  
```
    def get_htdocs_dirs(self):
        """
        Return a list of directories with static resources (such as style
        sheets, images, etc.)

        Each item in the list must be a `(prefix, abspath)` tuple. The
        `prefix` part defines the path in the URL that requests to these
        resources are prefixed with.
        
        The `abspath` is the absolute path to the directory containing the
        resources on the local file system.
        """
        from pkg_resources import resource_filename
        return [('nn', resource_filename(__name__, 'htdocs'))
```  
  
このreturn時に指定している'nn'という記号を使うことになる。  
たとえば、テンプレートファイル上で現在のプロジェクトのニコニコプラグインがインストールした画像を使用したい場合、以下のような記述になる  
  
```
<img src="${href.chrome('nn/images/worstMood.png')}" alt="Worst" title="最悪･･･"/></a>
```  
  
「nn」という記号をもとに実際のパスをつくっている。  
  
  
・genshiでテンプレートを書く場合、きちんと書かないとだめだ。たとえば「&lt;」と書くところを「&lt」と記述しても、多くのブラウザで正常にうごくが、genshiのテンプレートではエラーになる。  
  
・ディレクトリを値とするディレクトリをテンプレートで使用する場合、注意が必要だ。  
たとえば次のような判定が必要な場合があるとする  
  
```
 <img py:if = "mood_data[user][day] == 'worstMood'" src="${href.chrome('nn/images/worstMood.png')}" alt="最悪･･･"/>
```  
  
userまたは、dayのキーがない場合、エラーとなり落ちる。  
以下のように、ディレクトリを値とするディレクトリの場合、それぞれで存在チェックをすること。  
  
```
                <py:if test="mood_data[user]">
                  <py:if test="mood_data[user][day]">
                    <img py:if = "mood_data[user][day] == 'worstMood'" src="${href.chrome('nn/images/worstMood.png')}" alt="最悪･･･"/>
                  </py:if>
                </py:if>
```  
  
・Windows固有の問題だとおもうが、strftimeを使ってロケールにあった日付を使用すると正常に動作しないことがある。これは、帰ってくる文字がCP932であり、UTF8が前提としているTracと相性がわるい。  
今回はgetlocaleでロケールを確認して必要ならばUTF8に戻す処理をいれている  
  
```
    def getWeekStr(self, d):
      ret = d.strftime("%A")
      l,c = locale.getlocale()
      if(c=="932"):
        return ret.decode('cp932').encode('utf-8')
      else:
        return ret
```  
