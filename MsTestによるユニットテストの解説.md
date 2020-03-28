## 概要  
この記事ではVisualStudioインストール時に導入されるMsTestの使用方法について解説する。  
  
参考：  
  
 **単体テストの基本**  
https://msdn.microsoft.com/ja-jp/library/hh694602.aspx  
  
環境：  
VisualStudio Community 2013  
  
  
## 簡単なテストプロジェクト  
1.VisualStudioにてテストプロジェクトを追加する。  
  
![unittest1.png](/image/201e5953-d3d5-c87e-fe79-64645ce55608.png)  
  
  
2.UnitTestを記述する。  
  
**:UnitTest1.cs**  
```csharp:UnitTest1.cs
using System;
using Microsoft.VisualStudio.TestTools.UnitTesting;

namespace UnitTestSample
{
    [TestClass]
    public class UnitTest1
    {
        [TestMethod]
        public void TestMethod1()
        {
            int exp = 10;
            int act = 5 + 5;
            Assert.AreEqual(exp, act);
        }
    }
}

```  
  
3.テストを実行する  
![unittest2.png](/image/f0e971ea-ba18-8482-2c49-49037f02197d.png)  
  
4.テストエクスプローラーで結果を確認  
![unittest3.png](/image/c35d9f8c-afc0-bf4b-4c44-399045045dc3.png)  
  
## コマンドラインからのテストの実行  
継続的インテグレーションにMsTestを組み込む場合、コマンドラインにてMsTestを実行する必要がある。  
  
**注意**  
**下記の情報は古いです。Microsoft Fakesを使用するには「VSTest.Console.exe」を使用しましょう。**  
https://msdn.microsoft.com/ja-jp/library/ms182486.aspx  
  
  
```
# 環境変数の設定
"C:\Program Files (x86)\Microsoft Visual Studio 12.0\Common7\Tools\VsDevCmd.bat"

# テストの実行
mstest /testcontainer:UnitTestSample\UnitTestSample\bin\Debug\UnitTestSample.dll
```  
  
環境変数の設定で実行しているバッチファイルは、「開発者コマンド プロンプト for VS2013」で実行しているバッチと同じものである。  
  
mstestはテストを実行するコマンドである。  
/testcontainerのあとにテストプロジェクトが作成した、DLLを指定する。  
その他、MsTestの使用方法については下記のヘルプコマンドで確認のこと。  
  
```
mstest /help
```  
  
  
標準出力の出力例：  
  
  
```
Microsoft(R) Test Execution Command Line Tool Version 12.0.21005.1
Copyright (c) Microsoft Corporation. All rights reserved.

UnitTestSample\UnitTestSample\bin\Debug\UnitTestSample.dll を読み込んでいます...
実行を開始しています...

結果            トップ レベルのテスト
--            -----------
成功            UnitTestSample.UnitTest1.TestMethod1
1/1 テスト 成功

概要
--
テストの実行 完了 です。
  成功  1
  -----
  合計  1
結果ファイル:C:\Users\xxxx\Documents\Visual Studio 2013\Projects\Sample\TestResults\XXX-YYYY-PC 2015-05-26 17_59_47.trx
テストの設定:既定のテストの設定
```  
  
テストを実行することで、「TestResults」というフォルダがカレントディレクトリに作成されて、テスト結果とテストを行ったバイナリのコピーが格納される。  
この際、作成されたテスト結果は「.trx」拡張子のファイルとなり、VisualStudioを使用して開くことで内容を確認できる。  
  
![unittest4.png](/image/d5b0efba-e101-d1f0-8f31-89a026175b40.png)  
  
このリストには、すべての列が表示されているわけではない。  
列のヘッダを右クリックすることで、列の追加削除が行える。  
  
![unittest5.png](/image/d9a80722-2c6a-52d6-54d3-ea17e25b4764.png)  
  
また、この内容はXMLなので自前でパースしてExcelなどに結果を記述することも可能である。  
  
## テストエクスプローラーの機能  
テストエクスプローラーでは任意の方法でテストをグループ化できる。  
  
下記の例ではクラスごとにテストを分類したものである。  
  
![unittest14.png](/image/40c70ee2-b3a2-7b3e-5b51-4d880f52c4cd.png)  
  
また、Category属性を使用することで、各テストメソッドに特徴を指定でき分類が可能になる。  
  
```csharp
[TestMethod, TestCategory("カテゴリ1"), TestCategory("カテゴリ2")]
public void 足し算の確認()
{
}

[TestMethod, TestCategory("カテゴリ1")]
public void 引き算の確認()
{
}

[TestMethod, TestCategory("カテゴリ3")]
public void 常にオッケーのテスト()
{
}
```  
  
![unittest15.png](/image/6f630801-7bce-837b-96b8-b89ec16ae75d.png)  
  
  
  
その他、テストエクスプローラーの各機能については下記を参照。  
https://msdn.microsoft.com/ja-jp/library/hh694602.aspx#BKMK_Running_tests_in_Test_Explorer  
  
  
## 様々なAssert機能  
MsTestはAssertクラスで様々なAssertの方法を提供している。  
もっとも単純なものは以下のように期待値、結果、メッセージを付与するものである。  
  
```csharp
Assert.AreEqual(expected, actual, message)
Expected: 期待値
Actual:実際の値
Message:アサーションが失敗した時のメッセージ。テスト結果にでる
```  
  
このほかにも、浮動小数点をAreEqualする際に、許容範囲を指定できたり、Collectionに対しての検証も行える。  
  
  
```csharp
        [TestMethod]
        public void TestDouble()
        {
            double e = 1.5;
            double r = 1.51;
            Assert.AreEqual(e, r, 0.011); // 0.01だとNG
        }

       [TestMethod]
        public void TestCollection()
        {
            var expected = new List<int> { 1, 1, 2, 3, 4, 5};

            // 等価かどうか？
            // 2 つのコレクションが等価であるためには、同じ要素が同じ数だけ含まれている必要があります。要素の順番が一致している必要はありません。 2 つの要素が同一であるためには、その値が一致している必要があります。それぞれの要素が同一のオブジェクトを参照している必要はありません。
            CollectionAssert.AreEquivalent(expected, new List<int> { 1, 5, 4, 3, 2, 1 });

            // 同一であるか？
            CollectionAssert.AreEqual(expected, new List<int> { 1, 1, 2, 3, 4, 5 });

            // 一意であるか？
            CollectionAssert.AllItemsAreUnique(new List<int> { 1, 2, 3, 4, 5 });

            // 特定の値が含まれるか？
            CollectionAssert.Contains(new List<int> { 1, 2, 3, 4, 5 }, 3);

        }
```  
  
  
詳細は下記を参照。  
  
 **Assert クラス**   
https://msdn.microsoft.com/ja-jp/library/Microsoft.VisualStudio.TestTools.UnitTesting.Assert.aspx  
  
 **CollectionAssert クラス**   
https://msdn.microsoft.com/ja-jp/library/microsoft.visualstudio.testtools.unittesting.collectionassert.aspx  
  
## テストコードをステップ実行する方法  
下記のメニューをすることデバッグ実行が行える。  
「テスト」 > 「デバッグ」 > 「選択したテスト」　又は 「すべてのテスト」  
  
![unittest6.png](/image/8c8600f8-12e3-3fa7-319e-10e22f51396b.png)  
  
こうすることで、ブレイクポイントを指定した行で止めることができる。  
  
## テスト中の標準出力  
以下のようにテストメソッドで標準出力をしたとする。  
  
```csharp
        [TestMethod]
        public void TestStdOut()
        {
            Console.WriteLine("stdout {0} {1}", 1, "text");
        }
```  
  
この場合、テストエクスプローラーの結果で確認することができる。  
![unittest7.png](/image/058b6e5a-0fc6-0df4-e5b7-22c1fbcbc969.png)  
  
「出力」をクリックすることで、テスト結果の詳細が表示されて、標準出力の箇所に該当の文字が出力される。  
  
この内容は、コマンドラインで実行した場合に作成された「.trx」ファイルにも記録されている。  
  
**:テスト結果.trx**  
```xml:テスト結果.trx
<TestRun>
  <Results>
    ...
    <UnitTestResult ...>
      <Output>
        <StdOut>stdout 1 text</StdOut>
      </Output>
    </UnitTestResult>
  </Results>
</TestRun>
```  
  
 **出力ウィンドウ** には出力されないので注意すること。  
  
出力ウィンドウに出力したい場合は、Traceメソッドで該当のメッセージを記述して、Debug実行を行うこと。  
  
  
```csharp
        [TestMethod]
        public void TestStdOut()
        {
            System.Diagnostics.Trace.WriteLine("test");
        }
```  
  
![unittest8.png](/image/d1a53892-38e3-a8af-e022-286fb486d727.png)  
  
  
なお、Debugではない通常のテストを実行した場合、この出力は行われない。  
  
## テストケース共通の初期処理と終了処理  
テストクラスにおいて、各テストメソッドの初期処理と終了処理を共通化したい場合は、TestInitialize、TestCleanupなどを使用する。  
  
以下では、このコードには、メソッド、クラス、およびアセンブリの初期化とクリーンアップの実行順序を制御する属性を実験したものとなる。  
  
```csharp
using System;
using Microsoft.VisualStudio.TestTools.UnitTesting;

namespace UnitTestSample
{
    [TestClass]
    public class UnitTest2
    {
        [TestMethod]
        public void TestMethod1()
        {
            System.Diagnostics.Trace.WriteLine("TestMethod1");
        }

        [TestMethod]
        public void TestMethod2()
        {
            System.Diagnostics.Trace.WriteLine("TestMethod2");
        }

        [AssemblyInitialize]
        public static void AssemblyInit(TestContext context)
        {
            // アセンブリ内のすべてのテストが実行される前に、アセンブリによって取得されるリソースを割り当てるために使用されるコードを含むメソッドを識別します。 
            System.Diagnostics.Trace.WriteLine("AssemblyInit " + context.TestName);
        }

        [AssemblyCleanup]
        public static void AssemblyCleanup()
        {
            // アセンブリ内のすべてのテストが実行された後、アセンブリによって取得されたリソースを開放するために使用されるコードを含むメソッドを識別します。
            System.Diagnostics.Trace.WriteLine("AssemblyCleanup");
        }

        [ClassInitialize]
        public static void ClassInit(TestContext context)
        {
            // テスト クラス内の任意のテストが実行される前に、テスト クラスによって使用されるリソースを割り当てるために使用する必要のあるコードを含むメソッドを識別します。 
            System.Diagnostics.Trace.WriteLine("ClassInit " + context.TestName);
        }

        [ClassCleanup]
        public static void ClassCleanup()
        {
            // テスト クラスのすべてのテストが実行された後、テスト クラスによって取得されたリソースを解放するために使用されるコードを含むメソッドを識別します。
            System.Diagnostics.Trace.WriteLine("ClassCleanup");
        }

        [TestInitialize]
        public void TestInitialize()
        {
            // テスト クラスのすべてのテストに必要なリソースの割り当ておよび構成を行うために、テストの前に実行するメソッドを識別します。
            System.Diagnostics.Trace.WriteLine("TestInitialize");
        }

        [TestCleanup]
        public void TestCelean()
        {
            // テストが実行された後、テスト クラス内のすべてのテストによって取得されたリソースを開放するために使用される必要のあるコードを含むメソッドを識別します。 
            System.Diagnostics.Trace.WriteLine("TestCelean");
        }

    }
}
```  
  
デバッグ実行をした場合の、出力結果は以下のようになる。  
  
```
AssemblyInit TestMethod1
'vstest.executionengine.x86.exe' (CLR v4.0.30319: UnitTestAdapter: Running test): 'C:\Windows\Microsoft.Net\assembly\GAC_32\System.Transactions\v4.0_4.0.0.0__b77a5c561934e089\System.Transactions.dll' が読み込まれました。シンボルの読み込みをスキップしました。モジュールは最適化されていて、デバッグ オプションの [マイ コードのみ] 設定が有効になっています。
'vstest.executionengine.x86.exe' (CLR v4.0.30319: UnitTestAdapter: Running test): 'C:\Windows\Microsoft.Net\assembly\GAC_32\System.EnterpriseServices\v4.0_4.0.0.0__b03f5f7f11d50a3a\System.EnterpriseServices.dll' が読み込まれました。シンボルの読み込みをスキップしました。モジュールは最適化されていて、デバッグ オプションの [マイ コードのみ] 設定が有効になっています。
'vstest.executionengine.x86.exe' (CLR v4.0.30319: UnitTestAdapter: Running test): 'C:\Windows\Microsoft.Net\assembly\GAC_32\System.EnterpriseServices\v4.0_4.0.0.0__b03f5f7f11d50a3a\System.EnterpriseServices.Wrapper.dll' が読み込まれました。シンボルの読み込みをスキップしました。モジュールは最適化されていて、デバッグ オプションの [マイ コードのみ] 設定が有効になっています。
'vstest.executionengine.x86.exe' (CLR v4.0.30319: UnitTestAdapter: Running test): 'C:\Windows\Microsoft.Net\assembly\GAC_MSIL\System.Numerics\v4.0_4.0.0.0__b77a5c561934e089\System.Numerics.dll' が読み込まれました。シンボルの読み込みをスキップしました。モジュールは最適化されていて、デバッグ オプションの [マイ コードのみ] 設定が有効になっています。
ClassInit TestMethod1
TestInitialize
TestMethod1
TestCelean
TestInitialize
TestMethod2
TestCelean
ClassCleanup
AssemblyCleanup
```  
  
TestInitialize,TestCeleanはテストメソッド毎に実行されことが確認できる。  
  
  
## テストデータの利用  
MSTestではDataSource属性を使用することで、外部のファイルをテストデータとして使用できる。  
  
### 前提  
System.Dataが必要なので参照に追加しておくこと。  
  
![unittest10.png](/image/3bf063a5-f591-e23d-46e4-344977c62660.png)  
  
  
### もっともシンプルな例  
1.以下のようなtest.csvファイルを用意する。  
  
**:test.csv**  
```csv:test.csv
a,b,result
1,2,3
2,4,6
3,5,8
```  
  
 **このCSVはCP932で保存する**   
  
2.CSVをテストプロジェクトの直下に配置する。  
  
![unittest9.png](/image/87b71253-8589-e2da-7a32-1a813e9f6d65.png)  
  
3.追加したCSVのプロパティで、テストデータを「出力ディレクトリ―」にコピーするようにする。  
  
![unittest11.png](/image/9d9cc8a0-2a10-5b2b-572d-38aac86799d1.png)  
  
  
4.テストコードを記述する  
  
```csharp
        public TestContext TestContext { get; set; }

        [TestMethod]
        [DataSource("Microsoft.VisualStudio.TestTools.DataSource.CSV", @"test.csv", "test#csv", DataAccessMethod.Sequential)]
        public void TestCsv()
        {
            int a = (int)TestContext.DataRow["a"];
            int b = (int)TestContext.DataRow["b"];
            int result = (int)TestContext.DataRow["result"];
            System.Diagnostics.Trace.WriteLine("TestCsv " + a + " " + b + " " + result);
            Assert.AreEqual(result, a + b);
        }
```  
  
5.テストを実行すると、CSVを１行づつ順番によみこんでTestCsvを都度実行する。  
デバッグ実行をすると以下のような出力がされ、順次実行されていることがわかる。  
  
```
TestCsv 1 2 3
TestCsv 2 4 6
TestCsv 3 5 8
```  
  
### テストデータを格納するフォルダを指定する  
実際にテストデータを配置する場合、プロジェクトの直下ではなくフォルダを構成して配置する。  
  
![unittest12.png](/image/7b9a0c89-78d6-b59a-ab23-5708da7fde32.png)  
  
  
DataSource属性には、|DataDirectory|を指定して、その後、プロジェクトからの相対パスを記述する。  
  
```csharp
        [TestMethod]
        [DataSource("Microsoft.VisualStudio.TestTools.DataSource.CSV", @"|DataDirectory|\TestData\Test001\test.csv", "test#csv", DataAccessMethod.Sequential)]
        public void TestCsv()
        {
            int a = (int)TestContext.DataRow["a"];
            int b = (int)TestContext.DataRow["b"];
            int result = (int)TestContext.DataRow["result"];
            System.Diagnostics.Trace.WriteLine("TestCsv " + a + " " + b + " " + result);
            Assert.AreEqual(result, a + b);
        }
```  
  
### コマンドラインから実行した場合に、テストデータもコピーする。  
コマンドラインからMSTestを実行すると、テスト結果のOutフォルダにdllなどが出力される。  
この際、テストデータも出力したい場合は、DeploymentItem属性を使用する。  
  
```csharp
    [TestClass]
    [DeploymentItem(@"TestData\Test001\test.csv", @"TestData\Test001")]
    public class UnitTest1
    {
       // 略
    }
```  
  
これにより、Outフォルダにdllと共にテストデータが出力される。  
  
![unittest13.png](/image/e376ab01-a86b-ad84-a4a4-3e5e3014c55d.png)  
  
  
### ランダムアクセスによるデータの読み込み  
DataAccessMethod.Randomを実行することで、CSVをランダムな順番に読み込むことができる。  
  
  
```csharp
        [TestMethod]
        [DataSource("Microsoft.VisualStudio.TestTools.DataSource.CSV", @"|DataDirectory|\TestData\Test001\test.csv", "test#csv", DataAccessMethod.Random)]
        public void TestCsvRandom()
        {
            int a = (int)TestContext.DataRow["a"];
            int b = (int)TestContext.DataRow["b"];
            int result = (int)TestContext.DataRow["result"];
            System.Diagnostics.Trace.WriteLine("TestCsvRandom " + a + " " + b + " " + result);
            Assert.AreEqual(result, a + b);
        }
```  
  
デバッグ出力の例:  
  
```
TestCsvRandom 3 5 8
TestCsvRandom 1 2 3
TestCsvRandom 2 4 6
```  
  
参考：  
 **Data-driven troubles**   
http://www.codeproject.com/Articles/710072/Data-driven-troubles  
  
## 未確定なテスト項目  
現在、テストを一時的に無視したり、テストの成功・失敗の条件がわからない場合がある。  
この時は、コメントアウトをしないで、明確に無視するなり、未確定とすべき。  
  
### テスト項目を一時的に無視する  
ペンディングのテスト項目を実行させないで、警告を出力させたい場合がある。  
この際は、Ignore属性を使用する。  
  
![unittest16.png](/image/0a4fe898-e079-7ca1-5128-16c8662f1ef0.png)  
  
### 未確定な物  
Assert.Inconclusiveを使用する。  
  
```csharp
        [TestMethod]
        public void TestInconclusive()
        {
            Assert.AreEqual(1, 1);
            Assert.Inconclusive("Unable to determine success or failure");
        }

```  
  
この場合も、Ignoreと同様に警告として扱われる。  
  
## Privateメソッドのテスト  
Privateのメソッドについても、「PrivateObject」を利用することでテストが行える。  
  
```csharp
            var pbObj = new PrivateObject(_obj);
            var ret = pbObj.Invoke("privateAdd", 1, 2) as int?;
            Assert.AreEqual(3, ret);    
```  
  
PrivateObject クラス  
https://msdn.microsoft.com/ja-jp/library/ms245564(v=vs.110).aspx  
  
## 例外のテスト方法  
例外が発生する方法を確認する方法について説明する  
  
### ExpectedException属性を使用する  
ExpectedExceptionは以下のように使用する  
  
```csharp
        [TestMethod()]
        [ExpectedException(typeof(System.DivideByZeroException))]
        public void DivideTest()
        {
            int x = 1;
            int y = 0;
            int r = x / y;
            System.Diagnostics.Trace.WriteLine(r);
        }    
```  
  
System.DivideByZeroExceptionが発生した場合は成功、それ以外の場合は、テストは失敗する。  
この方法でテストした場合、１メソッドについて、１例外しか確認できない。  
  
### 自分でTry-catchして例外をテストする方法  
自分でTry-catchして例外をテストする場合は、例外が発生しなかった場合に、Assert.Failを利用してテストを失敗させる。  
  
 **ノーマルにMSTestを使おう**  
http://qiita.com/moonmile/items/269295ad5758fa69d203  
  
```csharp
        [TestMethod]
        public void DivideTest3()
        {
            try
            {
                int x = 0;
                int ans = 1 / x;
            }
            catch (System.DivideByZeroException ex)
            {
                // 例外が発生すればOK
                return;
            }
            // 別の例外が出た場合は、予期せぬ例外となって、テストの失敗となる。
            // 例外が発生ない場合は、以下のコードで例外を発生させる
            Assert.Fail("例外が発生しませんでした");
        }
```  
  
### 例外を投げるメソッドのテストを確認する関数を作る方法  
例外を投げるメソッドのテストを確認する関数を作り、確認することもできる。  
  
 **[Visual Studio 2012, C#] ユニットテストのメモ**   
http://fernweh.jp/b/vs2012-unittesting  
  
  
```csharp
        // http://fernweh.jp/b/vs2012-unittesting/#id-1
        private E GetException<E>(Action action) where E : Exception
        {
            try
            {
                action();
                return null;
            }
            catch (E ex)
            {
                return ex;
            }
        }

        [TestMethod()]
        public void DivideTest2()
        {
            var ex = GetException<Exception>(
                delegate() {
                    int x = 0;
                    int y = 1 / x;
                }
            );
            Assert.AreEqual("System.DivideByZeroException", ex.GetType().FullName);
        }
```  
  
## タイムアウトの設定  
タイムアウトは、個々のメソッドと全体に対して指定することができる。  
  
### 個々のテストメソッドにタイムアウトを指定する  
Timeoutにてミリ秒を指定することで、テストメソッド毎にタイムアウトを指定できる。  
テスト中に指定した期間を超えた場合、NGとなる。  
  
```csharp
        [TestMethod]
        [Timeout(2000)]  // Milliseconds
        public void TestTimeout1()
        {
            // NG
            System.Threading.Thread.Sleep(10000);
        }

        [TestMethod]
        [Timeout(TestTimeout.Infinite)]
        public void TestTimeout2()
        {
            // OK
            System.Threading.Thread.Sleep(10000);
        }
```  
  
### テスト全体のタイムアウトの指定  
テスト全体にタイムアウトを指定するには、テスト設定ファイルを作成する必要がある。  
まず、ソリューションの [ソリューション項目] フォルダーで、テストの設定ファイルを追加する。  
  
![unittest17.png](/image/b757baf5-c041-73c0-13f3-2c4ee581098f.png)  
  
テスト設定ファイルを作成するウィザードが起動するので、タイムアウトの設定がでるまで、既定の値で次に進む。  
  
![unittest19.png](/image/1db4a4f5-e075-cdaf-ce7d-5184592bb3b9.png)  
  
その後、ウィザードを終了すると、ソリューションにテスト設定ファイルが追加される。  
  
![unittest20.png](/image/2765373c-da53-d84c-7546-6a5f3b403d16.png)  
  
  
  
単体テストを実行する際に、この作成したテスト設定ファイルを適用して実行すると、タイムアウトが全体に適用される。  
  
VisualStudioでテストする場合、「テスト」→「テスト設定」→「テスト設定ファイルの選択」を指定できる。  
  
![unittest21.png](/image/b0c85bdd-dc78-fd3a-0928-824b950d79fa.png)  
  
  
コマンドラインから実行する場合に、テスト設定ファイルを指定するには「/testsettings」を使用する。  
  
```
MSTest /testcontainer:UnitTestSample\bin\Debug\UnitTestSample.dll　/testsettings:TestSettings1.testsettings
```  
  
  
参考：  
https://msdn.microsoft.com/ja-jp/library/ms243175.aspx  
  
## テスト結果に埋め込み可能な情報  
いくつかの属性を使用することで、テスト結果のXMLに情報を埋め込むことが可能である。  
  
### Description属性  
テストについての説明を指定するために使用される。  
  
```csharp
 [TestMethod]
 [Description("Test Case Description")]
 public void EnsureTestCaseValid()
 {      
 }
```  
  
### Owner属性  
テストの維持、実行、およびデバッグの担当者を指定するために使用する。  
  
```csharp
        [TestMethod]
        [Owner("mitagaki")]
        public void 常にオッケーのテスト()
        {
            Assert.AreEqual(true, true, message: "常時オッケー");
            Console.WriteLine("常時OK");
        }
```  
  
### Priority属性  
単体テストの優先順位を指定するために使用される。  
MSTestはテスト実行時に優先順位で絞り込むことが可能である。  
  
```csharp
        [TestMethod]
        [Priority(1)]
        public void TestPriority1()
        {
        }
```  
  
  
優先度の絞りこみには、minpriority,maxpriorityを用いる。  
  
```
MSTest /testcontainer:UnitTestSample\bin\Debug\UnitTestSample.dll /minpriority:2 /maxpriority:3
```  
  
## Moq  
単体テストを行う場合、クラス間の依存関係をなくすために、実際のオブジェクトの代わりにテスト用のオブジェクトを使用することがある。  
  
Moqは、このテスト用のオブジェクトを簡単に作成するためのライブラリである。  
https://github.com/Moq/moq4  
  
### インストール方法  
パッケージマネージャコンソールから下記のコマンドを実行  
  
```
PM> Install-Package Moq
```  
  
### 簡単なサンプル  
**:テスト対象**  
```csharp:テスト対象
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;


namespace CalcLib
{
    public interface IDataCalc
    {
        int GetData();
    }
    public class DataCalc : IDataCalc {
        virtual public int GetData() 
        {
            return 100;
        }
    }
    public class Calc
    {
        private IDataCalc _data;
        public Calc()
            : this(new DataCalc())
        {

        }

        public Calc(IDataCalc data)
        {
            _data = data;
        }

        public int IncData(int i)
        {
            return _data.GetData() + i;
        }
    }
}
```  
  
既定のDataCalc をモックを使用して実装する。  
モックで使うメソッドは、virtual でなければならない。  
  
**:テストコード**  
```cshar:テストコード
        [TestMethod]
        public void モックのテスト()
        {
            var mock = new Mock<CalcLib.DataCalc>();
            mock.Setup(c => c.GetData()).Returns(
               5
            );
            var con = mock.Object;
            var o = new CalcLib.Calc(con);
            Assert.AreEqual(10, o.IncData(5));
        }

```  
  
### Microsoft Fakes  
Microsoft Fakesでテストでのコードの分離がおこなえる。  
https://msdn.microsoft.com/ja-jp/library/hh549175.aspx  
  
しかし、Visual Studio Ultimate または Premiumが必要になる。  
  
詳細は下記を参照。  
http://qiita.com/mima_ita/items/9ebb0f40d3209f33a45d  
  
  
## コード カバレッジの計測  
Ultimate、または、Premiumを購入しないと無理。  
https://www.visualstudio.com/ja-jp/products/compare-visual-studio-products-vs.aspx  
  
コードカバレッジを100％目指そうとすると、まず頓挫するので、相応のコストを支払う覚悟がなければ気にしなくていいかもしれない。  
  
実際の使いかたは下記参照。  
http://qiita.com/mima_ita/items/05ce44c3eb1fd6e9dd46#テストコードで保障されていない箇所を調べてテストコードを書く  
  
## Pex  
Microsoftが開発中のPexを使用すると、検査対象のメソッドの単体テストのひな形を作成してくれる。  
  
http://research.microsoft.com/en-us/projects/pex/  
  
対象のメソッドを選択して「Generate Inputs /Outputs Table」を実行する。  
  
![unittest22.png](/image/9e936e87-b827-762a-f89b-86a168718b52.png)  
  
すると、そのメソッドに対して入力と結果の一覧が作成できる。  
![unittest23.png](/image/c313ef2e-93c1-6f6b-9801-0113ba35ba5e.png)  
  
これにより、単体テストのコードを作成したり、メソッドの内容を調べることが可能になる。  
しかし、VisualStudio2013時点では、自動生成された項目をユニットテストとして保存できないようなので、導入するなら2015が出るまで待った方が良いだろう。  
  
  
 **Using Pex and Microsoft Code Digger to Better Understand and Test Your Code**   
http://www.codeproject.com/Articles/583520/UsingplusPexplusandplusMicrosoftplusCodeplusDigger  
  
 **Visual Studio 2015 の新機能: Pex はユニットテストの福音となるか!?**   
http://www.slideshare.net/yasuhikoy/pex-44098704  
