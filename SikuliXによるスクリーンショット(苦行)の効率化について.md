## 目的  
SikuliX1.1.4ではスクリーンを監視して変更が生じたらスクリーンキャプチャをとることが容易に可能である。  
これにより、スクリーンキャプチャの苦行の労力を減らせるかを検討する。  
  
Sikulix1.1.4については下記参照  
https://github.com/mima3/note/blob/master/Sikulix1.1.4を使って画面の自動操作をする.md  
  
## コード  
  
```python
from datetime import datetime
import threading
import time
import sys
# 保存先のフォルダ
captureFolder = "C:\\tool\\sikulix\\log\\"

# 50ピクセル変更したらchangedを呼び出す。ここで調整する
changePixel = 50

threadWaitStopFunc = None
stopFlg = False


# 停止用のスレッド
def waitStopFunc():
    global stopFlg
    popup(u"キャプチャを停止する場合は [OK]を押下してください")
    stopFlg = True

# 範囲内の変更発生時のイベント
def changed(event):
    global stopFlg
    global threadWaitStopFunc
    print ('changed' + datetime.now().strftime('%Y%m%d_%H%M%S%f')[:-3])
    try:
        capture(selRegion, captureFolder, datetime.now().strftime('%Y%m%d_%H%M%S%f')[:-3])
    except:    
        print(sys.exc_info())
        stopFlg = True

# 範囲選択が表示されてRegionを選択する
selRegion = selectRegion(u"select caputure region.")
if selRegion is None:
    exit(1)

selRegion.onChange(changePixel, changed)

# 監視 alt-shift-c
selRegion.observeInBackground(FOREVER); 

threadWaitStopFunc = threading.Thread(target=waitStopFunc)
threadWaitStopFunc.start()

while True:
    if stopFlg:
        break
    wait(1)

popup(u'キャプチャが停止しました')

selRegion.stopObserver()
```  
  
※画像ファイルの作成に失敗した場合は「キャプチャが停止しました」というウィンドウが表示されるので、その場合は保存フォルダを見直す。  
  
## 使ってみる  
1. 実行すると矩形が選択できるので、監視したい範囲を選択する。  
![image.png](/image/ebc42396-0699-9bd6-dc32-2790c763bd32.png)  
  
2. 以下のポップアップが表示されるので**ＯＫを押さない**で邪魔にならない位置にどかしておく  
![image.png](/image/8f2b984a-d271-b552-4f5d-75806d48c3ee.png)  
監視を終了したくなったらＯＫをおして処理を停止する。  
  
3. 監視対象の画面を操作する。  
今回はコマンドプロンプトにて以下の操作を行った  
・dirコマンド実行  
・プロパティを表示  
  
4. ３の操作後に作成された画像ファイル  
![20190610_172512624.png](/image/1b22108a-fa04-161b-8dc0-5585b6e87656.png)  
![20190610_172513288.png](/image/921d7c16-8e79-86ed-c051-f78aa31d98ce.png)  
![20190610_172513620.png](/image/cab87192-d350-71f2-3497-48fc1c133102.png)  
![20190610_172514295.png](/image/8e077c0d-1df4-347a-594c-c244873a43bd.png)  
![20190610_172519636.png](/image/ac35e1c7-d6dd-fd6c-73f1-f66204399fa6.png)  
![20190610_172551705.png](/image/2e495eb0-6931-7b94-8924-4610ba0884f8.png)  
![20190610_172552031.png](/image/3a1bf8f7-a6d3-3b98-1e1d-8f72e21ebd53.png)  
![20190610_172552351.png](/image/5a9f9acc-4827-9e31-fdaa-61a91c7caacc.png)  
![20190610_172553022.png](/image/cae8b88a-ae28-e606-7cdf-3c687c49fcf7.png)  
![20190610_172553355.png](/image/7f4c2b41-34e1-1948-079c-1740cc942b8d.png)  
![20190610_172554021.png](/image/712c44e2-a172-7984-6e9c-d965ad9ec3e7.png)  
  
### 使用感  
・一文字づつのキー押下や、ウィンドウが表示される過程の半透明のものもキャプチャされてしまう  
・とはいえ、一枚一枚心を込めてスクリーンショットを撮る苦行よりはまし。  
　（あとで間引くか、大した量でないならそのまま納品でいいはず）  
・同様のことは動画でもできる。  
　ただ、画像で保存するアドバンテージとしては吹き出しつけたりの編集の容易性。  
  
**感動的なツールだな、だが、無意味だ。**※  
※SikuliX1.1.4は64bitのJavaが必要なので、こういうのが本当に必要な職場はどうせ、32ビットマシンしか支給されていないので使えない。仕方ないね。  
