# 複数プラットフォームをビルドしたい  
VisualStudio2019にて複数プラットフォームのビルドを一括でしたい場合は、構成マネージャでビルドをしたい構成を作成したのち、バッチビルドを行う。  
  
## 構成マネージャでの構成の追加方法  
(1)[ビルド]→[構成マネージャー]を選択  
![image.png](/image/cc99e59b-2acb-2bb6-6812-5461e626ee65.png)  
  
(2)アクティブソリューションプラットフォームのドロップダウンにて「新規作成」を選択  
![image.png](/image/17cfc639-f208-c783-f87b-e2a14206fe9a.png)  
  
(3)新しいプラットフォームダイアログにてプラットフォームを選択してOKを押す  
今回はx64とx86の両方を新規作成で追加した  
![image.png](/image/127c7991-0d10-04e1-d312-a5ee52d43c90.png)  
  
![image.png](/image/746ff7a3-8f2c-45c7-2ce6-48d147a754a7.png)  
  
(4)構成マネージャーで追加したプラットフォームが選択可能になる  
![image.png](/image/1d254ab7-7d35-431a-aa3c-fa90cdc998dc.png)  
  
(5)メインメニューのツールバーでも追加したプラットフォームが選択可能になる  
![image.png](/image/9f67c11d-eb48-606a-fb6a-fb4d6e1e514d.png)  
  
## バッチビルドの方法  
(1)[ビルド]→[バッチビルド]を選択  
![image.png](/image/eccd7dde-a1d7-9e81-e0b1-acc472c0b61b.png)  
  
(2)ビルドしたい構成にチェックを付けて「ビルド」または「リビルド」を実行  
![image.png](/image/367adf7b-3ac0-4974-a89a-ec88a284b3cc.png)  
  
(3)既定では以下のようなディレクトリにファイルが出力がされる。  
  
```
├─x64
│  └─Release
└─x86
    └─Release
```  
  
※出力先を変更したい場合は以下をいじる  
![image.png](/image/89524924-567a-4277-c6e4-0ab1c84e0abf.png)  
なおC#の場合、C++と違い$(SolutionDir)とかの環境変数を使用するとエスケープ処理がされるようなので、直接プロジェクトファイルを開いて直す必要がある模様。  
https://social.msdn.microsoft.com/Forums/vstudio/en-US/b0eb3746-285f-4ec6-9658-154f427cdb80/c-project-setting-output-directory-to-solution-dir?forum=msbuild  
  
### バッチビルドのメニューがない場合  
バージョンによってはバッチビルドのメニューがないこともある。その場合は下記を参考にメニューのカスタマイズでバッチビルドのメニューを追加する  
  
**VisualStdio2012でバッチビルドを行う方法**  
https://needtec.exblog.jp/21564296/  
  
以上  
