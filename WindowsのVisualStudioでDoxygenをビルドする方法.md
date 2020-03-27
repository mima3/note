この記事ではWindowsのVisualStudioを用いてDoxygenのソースファイルをコンパイルする方法について解説する。  
  
 **Compiling from source on Windows**   
http://www.stack.nl/~dimitri/doxygen/manual/install.html#install_src_windows  
  
  
 **環境**   
VisualStudio2008  
  
  
## 事前準備  
任意のソースコードをダウンロードして解凍する。  
https://codeload.github.com/doxygen/doxygen/zip/Release_1_8_9_1  
  
Doxygenのビルドにはflexを使用している。  
flexは字句解析のツールである。このツールはWindowsにも移植されている。  
http://sourceforge.net/projects/winflexbison/  
  
ダウンロードしたflexのフォルダにパスを通すか、doxygen-Release_1_8_9_1\winbuildにコピーする。  
  
その後、下記のような改名処理を行う。  
  
win_bison.exe→bison.exe  
win_flex.exe→flex.exe  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/ff4fa4fd-d663-9822-0f9d-9d575ef32fd3.png  
  
## Doxygenプロジェクトのみをビルド  
VisualStudioでソリューションを開いてDoxygenプロジェクトのみをビルドする。  
  
なお、他のプロジェクトは別の依存ライブラリが必要なので今回は対象外とする。  
  
・doxysearchにはxapianが必要  
http://stackoverflow.com/questions/25209217/missing-xapian-library-in-doxygen-build  
  
・doxywizardにはQt version 4が必要  
http://qt-project.org/  
  
### 文字コード関係のエラーが発生する場合...  
VisualStudioはBOMなしのUTF-8の挙動が怪しい場合がある。  
この場合は、プロジェクトで右クリックを押して、プロパティーを表示する。  
そして、「Language」にて「Use English Only」を選択すればよい。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/a63367c1-2fa0-1494-98e7-9e8fddb5fe7b.png  
  
Unicodeが必要な言語を全て排除できるので、円滑にデバッグを開始できる。  
  
## デバッグ実行  
デバッグ実行時にコマンドライン引数が与えられるので、問題のdoxyfileを指定してやれば、デバッグが可能になる。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/aa6b98a6-91b3-0898-fb4b-a0ef2f881f43.png  
