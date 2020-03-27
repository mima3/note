このドキュメントではVBAやVBScriptからCOM経由で.NETの処理を実行する方法について記述する。  
  
# 環境  
・Visual Studio2008 Pro  
・.NET Framework 2.0 を対象とする。  
・32ビットのOffice2010  
  
# .NETを使用したクラスモジュールをCOMにする方法  
  
１．Visual Studio 2008 を管理者権限で起動する。  
　これはCOM登録時にレジストリを変更するため。  
  
２．プロジェクトの追加で「.NET Framework 2.0」 と 「クラスライブラリ」を選択  
https://qiita-image-store.s3.amazonaws.com/0/47856/5c4a1c01-ea74-3c1a-1a18-d6b835d29390.png  
  
  
３. ビルドにてプラットフォームのターゲットを「x86」または「x64」とする。  
これは使用するExcelに合わせる。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/405f26bc-cf47-a003-f344-9083fe91e91e.png  
  
４. ビルドにてCOM相互運用機能の登録にチェックをつける。  
https://qiita-image-store.s3.amazonaws.com/0/47856/d72c34af-b0cb-e7b4-698d-9008eebf7706.png  
  
５. アプリケーションのアセンブリ情報で、「アセンブリをCOM参照可能にする」  
https://qiita-image-store.s3.amazonaws.com/0/47856/cedc7ece-5421-53eb-7b45-59ae3889acea.png  
  
６． 公開したいインターフェイスとその実態について以下のように記述する  
  
```csharp
using System.Runtime.InteropServices;
namespace NMeCabCom
{
    [ComVisible(true)]
    public interface INmcTagger
    {
        void Create();
        NmcNode[] Parse(string text);
    }
   
    [ClassInterface(ClassInterfaceType.None)]
    public class NmcTagger : INmcTagger
    {
        private MeCabTagger nmcab;
        public NmcTagger()
        {
        }

        public void Create()
        {
            MeCabParam p = new MeCabParam();
            p.DicDir = @"C:\dev\NMeCab\dic\ipadic";

            this.nmcab = MeCabTagger.Create(p);
        }

        public NmcNode[] Parse(string text)
        {
            List<NmcNode> result = new List<NmcNode>();
            if (this.nmcab == null)
            {
                return result.ToArray();
            }
            MeCabNode node = this.nmcab.ParseToNode(text);
            while (node != null)
            {
                result.Add(new NmcNode(node));
                node = node.Next;
            }
            return result.ToArray();
        }
   
    }
}
```  
  
まず、公開するインターフェイスの直前に「ComVisible(true)」と記述する。  
  
```csharp
    [ComVisible(true)]
    public interface INmcTagger
    {
        void Create();
        NmcNode[] Parse(string text);
    }
```  
  
そして、そのインターフェイスの実装部分はClassInterfaceType.Noneとする。  
  
```csharp
    [ClassInterface(ClassInterfaceType.None)]
    public class NmcTagger : INmcTagger
```  
  
  
７. ビルドを行うことでCOMの登録まで行われる  
  
# インターフェイスの設計時に注意すべきこと。  
 __・List<>などのジェネリック型はCOMとしては公開できない__   
  
 __・ClassInterfaceTypeにAutoDualを記述すると、インターフェイス不要にできるがすべきでない__   
   http://msdn.microsoft.com/ja-jp/library/ms182205.aspx  
  
 __・Uint、ULONG等のUnsignedの型をインターフェイスに使用してはいけない__   
COMとしては使用できても、VBAやVBSはUnsignedをサポートしていないので、ULONGなどのインターフェイスは使用できない。  
  
 __・オブジェクトの配列は返えしてはいけない__   
  
以下のようなCOMのインターフェイスがあったとする。  
  
```csharp
    public interface INmcTagger
    {
        string GetModulePath();
        void Create(NmcParam p);
        NmcNode[] Parse(string text);
    }
```  
  
このインターフェイスの場合、VBAやC#などの型を明示できるプログラミング言語では適切に処理できる  
  
```vbnet
Dim ret As NmcNode()
ret = t.Parse("This is a pen.")
```  
  
しかし、VBSの場合、型が明示できないため、Unknownとなり処理できなくなる。  
  
```vbnet
Dim ret ' As NmcNode()
ret = t.Parse("This is a pen.")
WScript.Echo TypeName(ret) ' Unknownとなり、以降処理できない。
```  
  
こういう場合は、配列を管理するインターフェイスを作成して、それを経由するようにする。  
  
```csharp
    public interface INmcNodeCollection
    {
        int Count { get; }
        NmcNode GetItem(int index);
    }
```  
  
配列ではないオブジェクトを返した場合は、VBSでも処理ができる  
  
```vbnet
set ret = t.Parse("This is a pen.")
For i = 0 To ret.Count- 1
  WScript.Echo TypeName(ret.GetItem(i))
  WScript.Echo ret.GetItem(i).Surface
Next
```  
  
# 利用方法  
## 開発環境以外の登録方法  
.NET Frameworkのフォルダに存在するregasmを使用する。  
作成したdllに対して /tlb と /codebase を付与してregasmを実行する。  
regasmは各バージョンの.NETに存在する。  
  
```
SET BIN=C:\Windows\Microsoft.NET\Framework\v2.0.50727

REM 登録
%BIN%\regasm  C:\dev\NMeCabCom\NMeCabCom\bin\Debug\NMeCabCom.dll /tlb /codebase
```  
  
この際、以下のメッセージが出力される場合がある。  
  
```
「RegAsm : warning RA0000 : 署名されていないアセンブリを /codebase を使用して登録すると、同じコンピュータにインストールされるその他のアプリケーションとの競合が生じる可能性があります。/codebase スイッチは署名されたアセンブリのみに使用できます。アセンブリに厳密な名前を付けて、再登録してください。」
```  
  
これは、警告だけであり、COMの登録はされている。  
この警告を消すには、プロジェクトの設定で「署名」→「アセンブリの署名」　厳密な名前のキーファイルを選択する必要がある。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/b057d5eb-b085-46ee-2b8f-ce1b901b3618.png  
  
  
依存するすべてのDLLが同様に厳密な名前を有しなければならないので気をつけること。  
  
### COMの削除方法  
/tlb と /unregister を付与したregasmコマンドを実行後、ファイルを消すればよい。  
  
  
```
regasm /tlb C:\dev\NMeCabCom\NMeCabCom\bin\Debug\NMeCabCom.dll  /unregister
```  
  
  
## ExcelVBAで利用する方法  
参照設定してやれば、使用できるようになる  
https://qiita-image-store.s3.amazonaws.com/0/47856/f501ff58-5826-38db-e82f-03270ac5298f.png  
  
以下のようにインテリセンスが効く。  
https://qiita-image-store.s3.amazonaws.com/0/47856/c57bf96d-37e9-a188-ff43-307ae7f77c16.png  
  
## VBSからの利用方法  
  
以下のようにCreateObjectを利用する。  
  
```
set t = CreateObject("NMeCabCom.NmcTagger")
```  
  
## 実際のサンプル  
 __VBAやVBScriptで形態素解析を行う方法__   
http://qiita.com/mima_ita/items/bc2aeb060ee12d280d7b  
