# まえがき  
どの現場にもあるTeraTermですが、マクロを実行することが可能です。  
https://ttssh2.osdn.jp/manual/ja/macro/  
  
TTLファイルにマクロを記述して、TeraTermのインストールディレクトリにあるttpmacro.exeから実行することでマクロが動作します。  
  
# 実行例  
## 特定のファイルを監視する  
tailコマンドで特定のファイルを監視するサンプルです。  
基本的に接続したり、コマンドを送信したらプロンプトの文字列の受信を待つことになります。  
  
コマンドの成否は「echo $?」を実行してエラーコードを読むことになると思いますが、waitだと「1」と「130」の区別がつかなかったのでwaitregexで行頭と単語の区切りを明確にしてチェックすることにしています。  
  
```
user = "username"

prompt = "["
strconcat prompt user
strconcat prompt "@centos7"

constr = '192.168.80.131:22 /ssh /auth=password /passwd=pass /user="
strconcat constr user
connect constr

; プロンプトが表示されるまで待機
wait prompt

logopen "C:\dev\teraterm\test.log" 0 1
logwrite 'Log start'#13#10

; コマンド実行
; https://ttssh2.osdn.jp/manual/ja/macro/command/index.html
sendln 'tail -f /home/username/test/test.txt'

; プロンプトが表示されるまで待機
wait  prompt

; 直前に終了された終了ステータス
; https://www.gnu.org/software/bash/manual/html_node/Special-Parameters.html#Special-Parameters
sendln "echo $?"

; waitだと1に必ず引っかかってしまう
; https://ttssh2.osdn.jp/manual/ja/reference/RE.txt
waitregex  "^130\b" "^1\b"
if result = 2 then
    messagebox "not found" "error"
endif

logclose

; 確認せずに閉じる
disconnect 0
```  
  
## ファイルの送受信  
[scpsend](https://ttssh2.osdn.jp/manual/ja/macro/command/scpsend.html)でファイルの送信、[scprecv](https://ttssh2.osdn.jp/manual/ja/macro/command/scprecv.html)でファイルの受信を行います。  
あくまで単一ファイルの送受信なのでフォルダを選択しても無駄です。  
  
```text
user = "username"

prompt = "["
strconcat prompt user
strconcat prompt "@centos7"

constr = '192.168.80.131:22 /ssh /auth=password /passwd=pass /user="
strconcat constr user
connect constr

; プロンプトが表示されるまで待機
wait prompt

; ファイル送信完了を確認する
SOURFILE = 'C:\dev\teraterm\send\test.txt'
DESTFILE = '~/test/test.txt'

sendln "rm ~/test/test.txt"
;; ファイル送信
scpsend SOURFILE DESTFILE
;; ファイル送信プロセス確認
do
  mpause 5000
  sprintf2 str 'ps -ef |grep -v grep |grep -c scp'
  sendln str
  waitln '0' '1'
loop while result != 1
;; ファイル送信が完了すると次のマクロを実行
sendln 'echo SCP finish'


scprecv '/home/username/squash-tm.war.back' "C:\dev\teraterm\rcv\tmp"

;; マクロ終了
end
```  
  
受信時は以下のダイアログが表示される。  
![image.png](/image/9fb9dd55-c095-cc0d-1ceb-eb681cb909fe.png)  
  
  
  
# あとがき  
単純なコマンド実行するだけならともかく、エラー処理等が面倒です。  
やるなら、シェルスクリプトを転送して、難しい処理はそこで実行して、結果を受信するとかの方が楽そうです。  
また、大量のファイルを送受信する場合はWinSCPの方が簡単そうです。  
  
