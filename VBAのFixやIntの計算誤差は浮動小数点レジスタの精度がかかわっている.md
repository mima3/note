## 問題  
Double型のデータをIntやFixで切り捨てた場合に誤差がでます。  
浮動小数点なので当然誤差は出ますが、CDblのキャストのタイミングでも誤差が発生します。  
  
ExcelVBAのイミデイトウィンドウで下記を実行してください。  
  
```vbnet
?Fix(0.6# * 10 )
5
?Fix(CDbl(0.6# * 10))
6
```  
  
  
環境によってはFix関数に渡す前にCDblでキャストした場合と、しない場合で計算結果が変わります。  
  
 **再現環境**   
　Office2010 32bit  
　Windows7 64biy  
　CPU Intel(R) Core(TM) i5 CPU M450 2.4GHz  
  
  
  
## 原因  
この問題は、下記のページでも議論されています。（このページではVB6が話題ですが、基本的に同じです）  
  
http://hanatyan.sakura.ne.jp/logbbs1/wforum.cgi?mode=allread&no=3276&page=0#3276  
  
この原因には、計算途中の浮動小数点の精度とDouble型の精度に違いがあるために発生しています。  
  
浮動小数点の演算の途中経過が64ビットの仮数を持つ80ビットのレジスタで計算しているが、Dobule型は53ビットの仮数を持つ64bitの浮動小数点なので、結果として誤差が表示してしまいます。  
  
 **「VC++とかで同じようなコードを書いても再現しないじゃないか！やっぱりVBAは壊れているじゃないか！（憤怒）」** と怒る兄貴、姉貴もおられると思いますが、浮動小数点レジスタの仮数を64ビットの仮数で計算するか、53ビットで計算するかはプログラム中で変更することができます。  
  
  
このことを確認するには、_controlfp_sの関数を実行して調べることができます。  
  
 **\_controlfp_s**   
http://msdn.microsoft.com/ja-jp/library/c9676k6h%28v=vs.80%29.aspx  
  
この関数から取得できる値に\_MCW_PCをマスクした結果で浮動小数点の精度が確認できます。  
  
 **\_MCW_PC (精度制御)**   
	  
|Mask|定数|value|  
|:---|:---|:----|  
|0x00030000|\_PC_24 (24 ビット)<BR>\_PC_53 (53 ビット)<BR>\_PC_64(64 ビット)|0x00020000<BR>0x00010000<BR>0x00000000|  
  
つまり、VC++のデフォルトでは、_MCW_PCの_PC_53がたっている状態であり、53ビットの仮数を持つ64bitの浮動小数点になっているため計算誤差がでないのです。  
  
### VBAでの浮動小数点レジスタの確認方法  
残念ながら、VBAで直接、\_controlfp_s関数を呼ぶことはできません。  
しかし、VBAではDLLを実行できるので、DLL経由で_controlfp_sを実行することで、確認することができます。  
  
まず、VC++でDLL側のコードを記述します。  
  
**DLL側のコード**  
```cpp:DLL側のコード
# include <windows.h>
# include <float.h>
int __stdcall GetControlFP() 
{ 
  int err,sts;
  err=_controlfp_s(&sts,0,0 );
  return sts;
} 
int WINAPI DllMain(HINSTANCE hInst, DWORD fdwReason, PVOID pvReserved) 
{ 
    return TRUE; 
} 
```  
  
  
 **VisualStdio2008SP1 で32bitでコンパイルした結果**   
http://needtec.sakura.ne.jp/vba/checkfloat/VBADll.zip  
  
※もし、64bitのVBAを使用している場合、64bitでビルドして64bitのDLLを作成する必要があります。  
  
  
作成したDLLをVBAから呼び出します。  
  
```vbnet
Option Explicit
' パスを修正すること
Declare Function GetControlFP Lib "C:\Users\ｘｘｘｘｘ\Documents\Visual Studio 2008\Projects\VBADll\Release\VBADll.dll" () As Long

Public Sub TestCon()
    '
    '  _MCW_PC (精度制御) 0x00030000
    '
    ' _PC_24 (24 ビット)　0x00020000
    ' _PC_53 (53 ビット)　0x00010000
    ' _PC_64 (64 ビット)　0x00000000
    Msgbox hex$(GetControlFP())
End Sub
```  
  
先にあげた検証環境では「C001F」という結果がかえってきました。  
これにより、精度制御が\_PC_64ビットであり、浮動小数点の演算の途中経過が64ビットの仮数を持つ80ビットのレジスタで計算していることが確認できました。  
  
## じゃあ、VBAたんは無罪なのね！  
・・・じつは、参照しているCOMやDLLなどにより変更される可能性があり、この結果は全ての状況で正しいとは限りません。  
  
たとえば、Jetを使うと途中の精度がかわったりするバグがあったりもします。  
http://support.microsoft.com/kb/308702/ja  
  
 **・・・やっぱり壊れているじゃないか!(憤怒)**   
