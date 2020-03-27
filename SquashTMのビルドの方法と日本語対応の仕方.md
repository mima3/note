# SquashTMとは  
SquashTMはWebベースのテスト管理ツールです。  
多言語化対応はしてますが、残念ながら日本語対応はしてません。  
また、当然の権利のように、文字化けしたりします。  
  
今回はSquashTMをビルドして文字化けを修正したり日本語対応の方法を調べたりします。  
  
# ビルドの方法  
公式にいくつか文章はありますが、古いです。  
  
すごく古い  
https://sites.google.com/a/henix.fr/wiki-squash-tm/developer/how-to-install-squashtm-project-into-eclipse  
※1.13以降はOSGi Frameworkはいらない。代りにSpringIDEがいる  
  
ちょっと古い  
https://bitbucket.org/nx/squashtest-tm/wiki/devguide/HowToInstallInIDE.md#!install-in-eclipse  
※1.19になるとJDK7はサポートしていないが、JDK7で動くような記載になっている。  
   
## 事前準備  
下記を用意します。  
・Java8以上  
・eclipse  
・mvn3.3以上  
・[ToroiseHg](https://tortoisehg.bitbucket.io/)　（Gitのような分散構成管理ツール）  
  
eclipceに入れるプラグインは以下の通り  
・springIDE  
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/1b11ca0f-8858-c883-aab8-c49d96a061d9.png  
  
・Groovy Development Tools  
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/c7c6127b-c3ef-cc57-8e62-5fde4c2cc303.png  
  
## ビルドと実行方法  
### ソースコードの取得からmvn installまで  
  
```txt
cd myeclipseworkspace
hg clone https://bitbucket.org/nx/squashtest-tm
cd squashtest-tm
mvn clean install -DskipTests -DskipITs
# 以下は不要の可能性がある
cd provision
mvn clean install -DskipTests
```  
  
#### com.mycila:license-maven-pluginでエラーが出る場合  
以下のようなエラーが発生する場合がある。  
  
```text
Failed to execute goal com.mycila:license-maven-plugin:2.11:check"
```  
  
この場合はルートフォルダで以下のコマンドを実行する。  
  
```
mvn license:format
```  
  
#### provisionでエラーがでる場合  
provisionのmvn installで以下のエラーが発生する場合がある。  
  
```
Non-resolvable parent POM for org.squashtest.tm:squash-tm-provision:[unknown-version]: Could not find artifact org.squashtest.tm:squash-tm:pom:1.19.0.RC3-SNAPSHOT and 'parent.relativePath' points at no local POM
```  
  
これはprovisionフォルダのpom.xml中のparentと親フォルダのpom.xmlの整合性が取れていない場合に発生する。  
2019/7/6時点では以下のような修正が必要だった  
  
```xml:parent/pom.xml
  <parent>
    <groupId>org.squashtest.tm</groupId>
    <artifactId>squash-tm</artifactId>
    <version>1.19.0.RELEASE</version> <<<<<< ここのVersionが親と食い違っていた
    <relativePath>../pom.xml</relativePath>
  </parent>
```  
  
### eclipseでの操作  
eclipseで下記の操作を行う  
#### プロジェクトのインポート  
1 メニューから[ファイル]>[インポート]を選択  
2 [Maven]>[既存のMavenプロジェクト]を背くん炊く  
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/76e23927-b00f-f1d3-198d-7687e241f55b.png  
3 「hg clone」で作成したフォルダを選択  
  
#### provisionモジュールをEclipseに導入  
この作業は最新では不要の可能性がある。  
  
1 メニューから[ウィンドウ]>[設定]を選択する。  
2 [プラグイン開発]>[ターゲット・プラットフォーム]を選択して「追加」ボタンを押下  
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/1facc8d8-c496-abb3-d760-44e03a165bef.png  
3 「空のターゲット定義で開始」を選択  
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/87c978a3-37ea-2677-2fa5-28810d17dc5a.png  
4 ターゲットコンテンツにて「追加」ボタンを押下  
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/e1a8cc9a-3fd7-c26d-b741-6b9aded975f9.png  
5 ディレクトリーを選択する。  
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/59809f07-cea0-fece-4cd9-c18ce1e78508.png  
6 ロケーションに「squashtest-tm/provision/target/eclipse-provision/bundles」を入力する。  
7 ターゲットコンテンツが追加されるので、引数タブでそれぞれ下記の値を入力する。  
プログラム引数 :  
  
```
-os ${target.os} -ws ${target.ws} -arch ${target.arch} -nl ${target.nl} -consoleLog -console
```  
  
**VM 引数 :**  
  
```
-Declipse.ignoreApp=true
-Dosgi.noShutdown=true
-Dorg.osgi.framework.system.packages.extra=com.sun.org.apache.xalan.internal.res,com.sun.org.apache.xml.internal.utils,
com.sun.org.apache.xpath.internal,com.sun.org.apache.xpath.internal.jaxp,com.sun.org.apache.xpath.internal.objects,com.sun.javadoc,
com.sun.tools.javadoc,javax.xml.namespace 
-Dbundles.configuration.location="${workspace_loc}/squashtest-tm/provision/target/config"
-Dorg.osgi.service.http.port=9090
-Dorg.osgi.service.http.port.secure=9443
```  
  
8. 完了後、今追加したターゲット定義にチェックを付ける  
  
#### Spring IDEプラグインの設定  
1 メニューから[実行]>[実行構成]を選択する  
2 実行構成画面にて[Spring Bootアプリケーション]を右クリックして「新規」ボタンを押下  
3 それぞれのタブで値を入力する  
**SpringBoot タブ**  
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/2e80c955-d815-41a9-ddf5-d3f36dd7851d.png  
  
・プロジェクト：tm.web  
・メイン型：org.squashtest.tm.SquashTm  
・プロファイル:h2,dev  
  
**引数 タブ**  
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/d39b5e5e-cfdf-06d3-e60e-565bcce9d789.png  
  
プログラム引数：-XX:MaxPermSize=256m -Xmx1024m  
  
**クラスパス タブ**  
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/d4c885d7-46ba-f44f-e5b9-fe7152dfbfed.png  
  
①「ユーザ・エントリー」を選択後、「拡張ボタン」を押下  
②「フォルダの追加」を選択  
③「tm.web/target/wro4j-spring-boot」を入力する  
  
#### 実行方法  
Spring IDEの設定で実行構成を設定しているので、そこで実行ボタンを押下する。  
その後、ブラウザから以下にアクセスする。  
(http://localhost:8080/squash  
  
##### 起動中にエラーになる場合の対応  
起動が失敗することがあった。この場合は以下を試してみた。  
なお、ログで例外が出ていても動作はしている模様  
  
1. データベースを一回消してみる  
・tm\data\squash-tm.mv.db  
・tm\data\squash-tm.trace.db  
  
2. 「mvn clean install」を行ってみる。  
  
## 色々修正してみる  
### 「Attach Test Cases」画面での文字化けの修正をしてみる  
テストスィートでテストケースを関連付ける際、日本語文字が文字化けする。  
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/9c8ace24-3698-bf02-dc2e-b6a44c6be460.png  
  
原因はjspにcontentTypeが設定していないため。  
以下のように修正する。  
  
```jsp:tm\tm.web\src\main\webapp\WEB-INF\jsp\page\campaign-workspace\show-test-suite-test-plan-manager.jsp
<%@ taglib prefix="authz" tagdir="/WEB-INF/tags/authz"%>

↓これを追加
<%@ page language="java" contentType="text/html; charset=utf-8" pageEncoding="utf-8"%>

<c:url var="testSuiteUrl" value="/test-suites/${ testSuite.id }" />
```  
  
結構化けているページがあるのでjspについてencodeが明示されているか見直した方がいいが、error.jspだけは、文字コードが明示されていたので、そのままのエンコードにしておいた。  
  
なお、全体的に日本語文字が化ける場合は、データベースを作成時にミスをしていてutf8に対応していないと考えられる。  
  
  
### 日本語メッセージを追加してみる。  
SquashTMはメッセージリソースをコピーすることで多言語対応が可能である。  
１．tm\tm.web\src\main\webapp\WEB-INF\messages\tm\messages.properties をコピーして、messages_ja.propertiesを作成する。  
２．Limyプロパティエディターで開く  
３．日本語に置き換える  
修正したら「tm-web」を「mvn install」したのち再起動。  
  
なお、言語の切り替えはブラウザの規定の言語によって行われる。  
  
### CentOS7の場合の置き換えと再起動方法  
作成したwarをサーバーに配置する方法は以下の通り  
  
1 tm/tm.web/target/tm.webXXXXXXX.warをsquash-tm.warに改名  
2 「sudo service squash-tm stop」でサービスをとめる  
3 CentOS7の場合は「 /usr/lib/squash-tm/bundles/」にコピー  
4 「sudo service squash-tm stop」でサービスを再開  
  
ちなみにログは以下にあるのでtail -f あたりで監視しとくといい。  
/var/log/squash-tm/squash-tm.log  
  
  
# さいごに  
これでSquashTMでバグがでても自力で直せるようになりました。  
バグがでても安心だな！（慢心）  
