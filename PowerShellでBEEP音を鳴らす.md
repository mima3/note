# 前書き  
PowerShellでBEEP音を鳴らすことができれば、自動処理中の結果を音で知らせることができます。  
今回はその方法を調べました。  
  
こんな感じになります。  
  
<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">古のテクニックBEEP音 <a href="https://t.co/oQM3vMHQga">pic.twitter.com/oQM3vMHQga</a></p>&mdash; m.ita (@mima_ita) <a href="https://twitter.com/mima_ita/status/1170630307101298688?ref_src=twsrc%5Etfw">September 8, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>  
  
  
# Beep音の鳴らし方  
WinAPIの[Beep](https://docs.microsoft.com/en-us/windows/win32/api/utilapiset/nf-utilapiset-beep)をC#経由で呼び出します。  
  
```powershell
$source = @"
using System;
using System.Runtime.InteropServices;

public static class WinApi
{
    [DllImport("kernel32.dll")]
    public static extern bool Beep(int freq,int duration);
}
"@
Add-Type -TypeDefinition $source
```  
  
  
## 実行例  
  
```powershell
	[WinApi]::Beep(440, 2000)
```  
  
第一引数に周波数、第二引数に継続するミリ秒を入力します。  
  
## 追記  
ConsoleオブジェクトがBeepサポートしていました（震え）  
  
```powershell
	[Console]::Beep(440,2000)
```  
  
  
# 音楽を鳴らしてみる。  
古のテクニックでBEEP音で音楽を鳴らすという技があります。  
下記のページに周波数と音階の対応表があるので利用します。  
  
**メロディのプログラム**  
http://web.archive.org/web/20190116021421/http://www.geocities.jp/shuinoue/myurobo/prog1.html  
  
  
```powershell
# http://web.archive.org/web/20190116021421/http://www.geocities.jp/shuinoue/myurobo/prog1.html
function global:Snd([string]$key, [int]$duration=100) {
    $KeySignature = @{
        "Fa-1"  = 294;
        "FaS-1" = 311;
        "So-1"  = 330;
        "SoS-1" = 349;
        "La-1"  = 370;
        "LaS-1" = 392;
        "Si-1"  = 415;
        "Do"  = 440;
        "DoS" = 466;
        "Re"  = 494;
        "ReS" = 523;
        "Mi"  = 554;
        "Fa"  = 587;
        "FaS" = 622;
        "So"  = 659;
        "SoS" = 699;
        "La"  = 740;
        "LaS" = 784;
        "Si"  = 831;
        "Do+1"  = 880;
        "DoS+1" = 932;
        "Re+1"  = 988;
        "ReS+1" = 1047;
        "Mi+1"  = 1109;
        "Fa+1"  = 1175;
        "FaS+1" = 1245;
        "So+1"  = 1319;
        "SoS+1" = 1397;
        "La+1"  = 1480;
        "LaS+1" = 1568;
        "Si+1"  = 1661;
        "Do+2"  = 1760;
        "DoS+2" = 1865;
        "Re+2"  = 1976;
        "ReS+2" = 2093;
        "Mi+2"  = 2218;
        "Fa+2"  = 2349;
        "FaS+2" = 2489;
        "So+2"  = 2637;
        "SoS+2" = 2794;
        "La+2"  = 2960;
        "LaS+2" = 3136;
        "Si+2"  = 3322;
    }
    #
    $freq = $KeySignature[$key]
    $ret = [WinApi]::Beep($freq, $duration)
}
```  
  
以下のように鳴らします。  
  
```powershell

Snd So-1 300;Snd Do 300;Snd Mi 300;Snd So 300;Snd Do+1 300;Snd Mi+1 300;Snd So+1 600;Snd Mi+1 600;
Snd SoS-1 300;Snd Do 300;Snd ReS 300;Snd SoS 300;Snd Do+1 300;Snd ReS+1 300;Snd SoS+1 600;Snd ReS+1 600;
Snd LaS-1 300;Snd Re 300;Snd Fa 300;Snd LaS 300;Snd Re+1 300;Snd Fa+1 300;Snd LaS+1 900;
Snd Si+1 300;Snd Si+1 300;Snd Si+1 300;Snd Do+2 1200;
```  
  
# まとめ  
今回はBEEP音をならしてみました。  
簡単な警告音ならともかく、令和の時代なので音楽は別の方法で鳴らした方がいいと思った（こなみかん）。  
