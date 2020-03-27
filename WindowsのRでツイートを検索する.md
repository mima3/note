## 背景  
twitteRを用いればR言語でツイートを検索できますが、以下のページに記述した通り、現行リリースされているバージョンではくっそ面倒です。  
  
http://needtec.exblog.jp/20588155/  
  
また、現在、パッケージとしてリリースされている1.1.7バージョンではいくつかのバグがあります。  
https://github.com/geoffjentry/twitteR/blob/master/NEWS  
  
・UTF8が文字化けする  
・ユーザIDが64ビット対応していない  
  
ここではGitHub中の最新コードを使用して日本語の文字を検索してみます。  
https://github.com/geoffjentry/twitteR/  
  
 **注意**   
2015-02-11に1.1.8がリリースされたので、Githubからでなくても大丈夫になっているはずです。  
http://cran.r-project.org/web/packages/twitteR/index.html  
  
## 環境  
Windows7  
R3.1.2　RGuiで行う  
ライブラリは下記のフォルダに格納するものとする。  
C:\R31_Package  
  
## 手順  
1.依存するパッケージと、Githubからインストールするツールをインストールします。  
  
```
install.packages(c("devtools", "rjson", "bit64", "httr"),"C:\\R31_Package")
```  
  
2.GithubからTwitteRをインストールします  
  
```
.libPaths("C:\\R31_Package") #ライブラリのパスを指定
library(devtools) #警告でますが無視してインストールできました。
install_github("twitteR", username="geoffjentry")
```  
  
3.ツイートを検索します  
  
```
library(twitteR)
setup_twitter_oauth("Consumer Key","Consumer Secret ","Access Token","Access Token Secret") # ローカルファイルにキャッシュするか聞かれます。どっちえらんでも動きます。
searchTwitter(iconv("おっぱい","CP932","UTF-8")) #日本語の場合はUTF8に変換する

```  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/657e17a1-4fcc-e54a-e7ce-fed716a34aaf.png  
  
コードを見る限り、since、untilもサポートしているので、昔にさかのぼって取得することもできるかと思います（未検証）  
  
https://github.com/geoffjentry/twitteR/blob/29f574b0ab6f8760b1471d12a4e70245731c4848/R/search.R  
  
## 絵文字が入っている場合の対応  
Twitterの公式クライアントでは機種依存の絵文字が入っている場合、サーバー側で画像に変換して表示して返しています。  
対して、TwitterAPIでは、絵文字が入っている場合、その文字コードをそのまま返します。おそらく、これはUnicodeの外字領域を使っており、環境によっては文字として表示できません。  
  
さいわい、iconvではsub引数を用いて、この変換できない文字を置き換えることができます。  
以下がUTF8に変換できない文字を削除する例になります。  
  
```
# 絵文字が入るように検索
ret=searchTwitter("needtec.sakura.ne.jp");
for(r in ret) {
  print(iconv(r$text, "UTF-8", "UTF-8", ""))
}
```  
