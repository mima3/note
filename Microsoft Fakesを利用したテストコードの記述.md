# Microsoft Fakesとは？  
本来、プログラムコードはまっとうなプログラマーが、まっとうなスケジュールで設計、実装すれば自然とテストしやすくなります。  
  
しかしながら、我々は楽園には住んでいないので、さまざまな理由で名状しがたきプログラムコードは作成され、厳しい選択肢を突き付けられます。  
  
選択①　トムデマルコのような管理者がプログラムを修正するようなスケジュールを引き直す。  
選択②　オブジェクト指向を理解した増員が来て助けてくれる。  
選択③　直せない。現実は非情である。  
  
Microsoft Fakesはこのような非情な現実を突き付けられた者にとっての一助になります。  
  
MicrosoftFakesのShim機能を使用することで、複雑に依存した機能を切り離して特定の箇所のみテストを実施することが可能になります。  
  
![unit004.png](/image/270e376f-8853-db30-5ee0-1617c97dd9b0.png)  
  
**Microsoft Fakes を使用したテストでのコードの分離**  
https://msdn.microsoft.com/ja-jp/library/hh549175.aspx  
  
たとえば、データベースや別システムに通信するような機能のテストであっても、データベースや別システムがテストに都合のいいデータを返すように偽装できます。  
  
たとえば、現在時刻に依存したテスト対象や、ユーザ名に依存したものでもテストに都合のいい現在時刻やユーザ名を偽装して返すことができます。  
  
これにより、テスト対象に手を加えることなくテストを実施することが可能になります。  
  
ただし、VisualStudio2013ではPremiume以上、VisualStudio2015ではEnterpriseのエディションが必要になります。  
  
また基本的なVisualStudioでのテストコードの書き方は下記を参照してください。  
http://qiita.com/mima_ita/items/55394bcc851eb8b6dc24  
  
# Microsoft Fakesを試す  
## 環境構築  
VisualStudioEnterpriseは下記のページから無料試用版を取得することができます。  
https://www.visualstudio.com/ja/downloads/  
  
なお、75万円くらいするのでなかなかハードルが高いです。  
  
  
## 簡単なチュートリアル  
(1)以下のような構成のプロジェクトを用意する  
![unit001.png](/image/333cbecf-2587-023b-bc60-124509d737eb.png)  
  
|プロジェクト名|説明|  
|:--|:--|  
|ClassLibrary1|テスト対象|  
|UnitTestProject1|テストプログラム|  
  
**:テスト対象**  
```csharp:テスト対象
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace ClassLibrary1
{
    public class Class1
    {
        private int a;
        public Class1(int a)
        {
            this.a = a;
        }
        public int Cal(int x, int y)
        {
            return a + x + y;
        }

    }

    public class Class2
    {
        Class1 c1 = new Class1(5);
        public int CallCal()
        {
            return c1.Cal(1, 2);
        }
    }
}
```  
  
(2)テストプロジェクトの参照から偽装したいアセンブリを選択して右クリックをして「Fakesアセンブリに追加」を実行する。  
![unit002.png](/image/cdc4677f-6c49-25b2-2ec5-72fa3ff749d7.png)  
  
(3)Fakesフォルダに拡張子がfakesのファイルが作成される。  
![unit003.png](/image/a8eb4b5f-c9c3-0d5c-aab2-a3330916e5e9.png)  
  
構成管理にあげる場合は、このfakesファイルを上げること。  
  
(4)テストコードの実装  
  
**:テストコードの例**  
```csharp:テストコードの例
using System;
using Microsoft.VisualStudio.TestTools.UnitTesting;
using Microsoft.QualityTools.Testing.Fakes;

namespace UnitTestProject1
{
    [TestClass]
    public class UnitTest1
    {
        [TestMethod]
        public void TestMethod1()
        {
            // ShimsContextのブロック内のみ偽装する。
            using (ShimsContext.Create())
            {
                var c2 = new ClassLibrary1.Class2();

                ClassLibrary1.Fakes.ShimClass1.AllInstances.CalInt32Int32 = (ClassLibrary1.Class1 obj, int x, int y) =>
                {
                    // c1.Cal(1, 2); が実行されていることを確認する
                    Assert.AreEqual(c2.c1, obj);
                    Assert.AreEqual(1, x);
                    Assert.AreEqual(2, y);

                    return 999999;
                };

                var act = c2.CallCal();

                // Shimで偽装した値が返ってくることを確認する。
                Assert.AreEqual(999999, act);
            }
        }
    }
}
```  
  
(5)テストを実施するとShimで偽装した値が取得できることが確認できる。  
  
## Shimの使い方  
  
### Shimの対象  
public/private/protectedのあらゆるスコープのメソッドを偽装できるが、privateの内部クラスや型をパラメータもしくは戻り値にする関数は偽装はできない。  
  
**:偽装対象のクラス**  
```csharp:偽装対象のクラス
    public class Class3
    {
        public int test1()
        {
            return 1;
        }

        public static int test2()
        {
            return 2;
        }
        private int test3()
        {
            return 3;
        }
        protected int test4()
        {
            return 5;
        }

        public class Test5Ret
        {
        }

        private Test5Ret test5()
        {
            return new Test5Ret();
        }

        private class Test6Ret
        { }

        // パラメータまたは戻り値が公開された型でないので偽装ができない。
        private Test6Ret test6()
        {
            return new Test6Ret();
        }
    }
```  
  
**:Shimによる偽装の方法**  
```csharp:Shimによる偽装の方法
        [TestMethod]
        public void TestMethod4()
        {
            // ShimsContextのブロック内のみ偽装する。
            using (ShimsContext.Create())
            {
                var c2 = new ClassLibrary1.Class2();

                ClassLibrary1.Fakes.ShimClass3.AllInstances.test1 = (ClassLibrary1.Class3 obj) =>
                {
                    return 11;
                };

                ClassLibrary1.Fakes.ShimClass3.test2 = () =>
                {
                    return 22;
                };

                ClassLibrary1.Fakes.ShimClass3.AllInstances.test3 = (ClassLibrary1.Class3 obj) =>
                {
                    return 33;
                };

                ClassLibrary1.Fakes.ShimClass3.AllInstances.test4 = (ClassLibrary1.Class3 obj) =>
                {
                    return 44;
                };

                ClassLibrary1.Fakes.ShimClass3.AllInstances.test5 = (ClassLibrary1.Class3 obj) =>
                {
                    return null;
                };

                //これは作られない。
                //ClassLibrary1.Fakes.ShimClass3.AllInstances.test6 = (ClassLibrary1.Class3 obj) =>
                //{
                //    return null;
                //};


            }
        }
```  
  
  
### Shimの関数名の作成ルール  
偽装するための関数は以下の命名規則で作成される。  
  
**インスタンスのメソッドの場合：**  
【テスト対象の名前空間】.Fakes..Shim【クラス名】.AllInstances.【関数名】【パラメータ1の型】【パラメータ2の型】..【パラメータnの型】(クラスのインスタンス,パラメータ1,パラメータ2...)  
  
**スタティックのメソッドの場合：**  
【テスト対象の名前空間】.Fakes..Shim【クラス名】..【関数名】【パラメータ1の型]【パラメータ2の型】..【パラメータnの型】(パラメータ1,パラメータ2...)  
  
**プロパティの場合：**  
【テスト対象の名前空間】.Fakes..Shim【クラス名】.AllInstances.Get【プロパティ名】(クラス)  
【テスト対象の名前空間】.Fakes..Shim【クラス名】.AllInstances.Set【プロパティ名】【パラメータの型】(クラスのインスタンス,パラメータ)  
  
  
偽装対象の関数のパラメータ数分だけ名称がながくなるので注意が必要。  
古いVS2012だと長すぎると偽装用の関数が作成されない。（256あたり？）  
VS2015だと、後方の文字を切って適正な文字に変換しているようだ。  
  
### 現在日付の偽装  
現在日付はテストをするうえでやっかいだが、これも偽装できる。  
  
(1)System.dllのfakesアセンブリを追加する。  
![unit005.png](/image/98bb1768-632a-872f-58a5-3c8bd6846fbd.png)  
  
![unit006.png](/image/bc00a29e-9971-272f-6b19-f4d9950d9fce.png)  
  
  
(2)System.DateTime.Nowを偽装する。  
  
```csharp
        [TestMethod]
        public void TestMethod2()
        {
            // ShimsContextのブロック内のみ偽装する。
            using (ShimsContext.Create())
            {
                DateTime exp = new DateTime(2000, 10, 5);
                System.Fakes.ShimDateTime.NowGet = () =>
                {
                    return exp;
                };

                var act = System.DateTime.Now;

                Assert.AreEqual(exp, act);
            }
        }
```  
  
### 現在ユーザやマシン名などのSysetem.Environmentを偽装する。  
(1)Fakesフォルダの「mscorlib.fakes」を開く  
![unit007.png](/image/4b9a303c-a086-01a0-55b4-7bd3786e1319.png)  
  
(2)ShimGenerationにSystem.Environmentを追加する。  
  
```xml
<Fakes xmlns="http://schemas.microsoft.com/fakes/2011/">
  <Assembly Name="mscorlib" Version="4.0.0.0"/>
  <ShimGeneration>
    <Add FullName="System.Environment"/>
  </ShimGeneration>  
</Fakes>
```  
(3)リビルドを行う。  
  
(4)下記のように偽装を行う。  
  
```csharp

        [TestMethod]
        public void TestMethod3()
        {
            // ShimsContextのブロック内のみ偽装する。
            using (ShimsContext.Create())
            {
                System.Fakes.ShimEnvironment.UserNameGet = () =>
                {
                    return "user";
                };
                System.Fakes.ShimEnvironment.MachineNameGet = () =>
                {
                    return "machine";
                };

                Assert.AreEqual("user", Environment.UserName);
                Assert.AreEqual("machine", Environment.MachineName);
            }
        }
```  
  
### 内部クラスのメソッドを偽装  
内部クラスはShim親クラス名.Shim内部クラス名という形で偽装が可能である。  
  
**:内部クラス**  
```csharp:内部クラス
    public class Class5
    {
        public class Class5Inner
        {
            public int Test5Inner()
            {
                return 5;
            }
        }
    }
```  
  
**:内部クラスのメソッドを偽装**  
```csharp:内部クラスのメソッドを偽装
        [TestMethod]
        public void TestMethod5()
        {
            // ShimsContextのブロック内のみ偽装する。
            using (ShimsContext.Create())
            {
                ClassLibrary1.Fakes.ShimClass5.ShimClass5Inner.AllInstances.Test5Inner = (ClassLibrary1.Class5.Class5Inner obj) =>
                {
                    return 99;
                };
                var o = new ClassLibrary1.Class5.Class5Inner();
                Assert.AreEqual(99, o.Test5Inner());
            }
        }
```  
  
### ベースクラスのメソッドを偽装  
ベースクラスの偽装はベースクラス自体を偽装する必要がある。継承先のShimにはベースクラスのメソッドは存在しない。  
  
**:テスト対象**  
```csharp:テスト対象
    public class Class6Base
    {
        protected int Test6Base()
        {
            return 6;
        }
    }

    public class Class6: Class6Base
    {
        public int Test6BasePlus1()
        {
            return base.Test6Base() + 1;
        }
    }
```  
  
**:テストコード**  
```csharp:テストコード
        [TestMethod]
        public void TestMethod6()
        {
            // ShimsContextのブロック内のみ偽装する。
            using (ShimsContext.Create())
            {
                ClassLibrary1.Fakes.ShimClass6Base.AllInstances.Test6Base = (ClassLibrary1.Class6Base obj) =>
                {
                    return 99;
                };
                var o = new ClassLibrary1.Class6();
                Assert.AreEqual(100, o.Test6BasePlus1());
            }
        }
```  
  
### ジェネリックメソッドの偽装方法  
ジェネリックメソッドの偽装方法は型を指定したShimを作成する必要がある。  
  
**:テスト対象**  
```csharp:テスト対象
    public static class Class7
    {
        /// <summary>
        /// xとyを入れ替える
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="x"></param>
        /// <param name="y"></param>
        static void Swap<T>(ref T x, ref T y)
        {
            T temp;
            temp = x;
            y = x;
            x = temp;
        }

        public static void Test7()
        {
            int x = 1;
            int y = 9;
            Swap(ref x, ref y);
        }
    }
```  
  
**:テストコード**  
```csharp:テストコード
        [TestMethod]
        public void TestMethod7()
        {
            // ShimsContextのブロック内のみ偽装する。
            using (ShimsContext.Create())
            {
                bool callSwap = false;
                ClassLibrary1.Fakes.ShimClass7.SwapOf1M0RefM0Ref<int>((ref int x, ref int y) =>
                {
                    // 
                    callSwap = true;
                    Assert.AreEqual(1, x);
                    Assert.AreEqual(9, y);
                    return;
                });
                ClassLibrary1.Class7.Test7();
                Assert.AreEqual(true, callSwap);
            }
        }
```  
  
### ジェネリッククラスの偽装  
ジェネリッククラスのShimを使うにはShimクラス名<型>を定義する必要がある。  
  
**:偽装対象**  
```csharp:偽装対象
    public class Class8<K, V>
    {
        public K key {set; get;}
        public V value { set; get; }

        public void Log()
        {
            Console.WriteLine(key.ToString() + value.ToString());
        }
    }
```  
  
**:テストコード**  
```csharp:テストコード
       [TestMethod]
        public void TestMethod8()
        {
            // ShimsContextのブロック内のみ偽装する。
            using (ShimsContext.Create())
            {
                bool callLog = false;

                ClassLibrary1.Fakes.ShimClass8<int,string>.AllInstances.Log = (ClassLibrary1.Class8<int,string> obj) => 
                {
                    // 
                    callLog = true;
                };
                var o = new ClassLibrary1.Class8<int, string>();
                o.Log();
                Assert.AreEqual(true, callLog);
            }
        }
```  
  
## どう利用するか？  
Shimは強力なツールであります。これにより多くのコードを通すことが可能になり、カバレッジも100%近くにすることができるかもしれません。  
  
しかし、適切に実行が検証されないテストは、無意味です。  
ここでは、この強力なツールをどう利用してコードの検証を行うかを考えてみます。  
  
### パラメータやShimを実行した際のオブジェクトの状態を確認するようにする。  
Shimのパラメータをチェックすることで、渡された値が期待値通りか？実行時のオブジェクトが期待通りの状態か検査すべきです。  
  
```csharp
            // ShimsContextのブロック内のみ偽装する。
            using (ShimsContext.Create())
            {
                ClassLibrary1.Fakes.ShimClass1.AllInstances.CalInt32Int32 = (ClassLibrary1.Class1 obj, int x, int y) =>
                {
                    // c1.Cal(1, 2); が実行されていることを確認する
                    Assert.AreEqual(1, x);
                    Assert.AreEqual(2, y);
                    // 実行時のClass1.xxxxが123であることを確認する
                    Assert.AreEqual(123, obj.xxxx);

                    return 999999;
                };
            }
```  
  
### 実行回数または実行されたか否かの検査を行う  
Shimの実行回数をカウントとするか、フラグで管理することによりShimが期待の回数実行されるかどうかを確認します。  
  
```csharp
        public void TestMethod10()
        {
            // ShimsContextのブロック内のみ偽装する。
            using (ShimsContext.Create())
            {
                int callCalCnt = 0;
                ClassLibrary1.Fakes.ShimClass1.AllInstances.CalInt32Int32 = (ClassLibrary1.Class1 obj, int x, int y) =>
                {
                    // 実行した回数を数える
                    ++callCalCnt;
                    return 999999;
                };

                var c2 = new ClassLibrary1.Class2();
                var act = c2.CallCal();
                Assert.AreEqual(1, callCalCnt, "c1.Calが1回実行されていることを確認");
            }
        }
```  
  
### 複数回実行されるShimの場合  
Shimが複数回実行される場合は、以下のようにリストで期待するパラメータとShimが返す値を設定します。  
  
**:テスト対象**  
```csharp:テスト対象
    public class Class11
    {
        private int x = 0;
        private int Inc(int i)
        {
            x += i;
            return x;
        }

        public int Test11()
        {
            int i = 0;
            i = Inc(5);
            i = Inc(i);
            i = Inc(i);
            return i;
        }
    }
```  
  
**:テスト対象**  
```csharp:テスト対象
        class ShimIncData
        {
            public int expParam { set; get; }
            public int returnVal { set; get; }
        }

        [TestMethod]
        public void TestMethod11()
        {
            using (ShimsContext.Create())
            {
                int callIncCnt = 0;
                var incShimDataList = new List<ShimIncData>();
                incShimDataList.Add(new ShimIncData
                {
                    expParam = 5,
                    returnVal = 6
                });
                incShimDataList.Add(new ShimIncData
                {
                    expParam = 6,
                    returnVal = 7
                });
                incShimDataList.Add(new ShimIncData
                {
                    expParam = 7,
                    returnVal = 8
                });

                ClassLibrary1.Fakes.ShimClass11.AllInstances.IncInt32 = (ClassLibrary1.Class11 obj, int i) =>
                {
                    // 実行した回数を数える
                    int ix = callIncCnt;
                    ++callIncCnt;

                    Assert.AreEqual(incShimDataList[ix].expParam, i);

                    // 
                    return incShimDataList[ix].returnVal;
                };

                var o = new ClassLibrary1.Class11();
                var act = o.Test11();

                Assert.AreEqual(3, callIncCnt, "Incが3回実行されること");
                Assert.AreEqual(8, act);
            }
        }
```  
  
今回は回数で返す値を替えたが、分岐を作ってパラメータが～だったら～を返すという方法でもいいでしょう。  
  
# まとめ  
Microsoft Fakesを使用するとテスト対象の依存先のコードを偽装することができます。  
これにより、テスト対象に手を加えずにテストコードを記述することができます。  
  
  
