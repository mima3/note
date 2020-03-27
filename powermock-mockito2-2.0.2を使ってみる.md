# はじめに  
以前でJavaの[jmockit](https://qiita.com/mima_ita/items/4e48a24561960851e3fa)を調べてみたんですが、ちょっと合わないのでプランBとしてMockitoとPowermockを調べた際のメモになります。  
  
**Mockito**  
https://site.mockito.org/  
  
**Mockito JavaDoc**  
https://javadoc.io/doc/org.mockito/mockito-core/3.1.0/org/mockito/Mockito.html  
  
**Powermock**  
https://github.com/powermock/powermock/  
  
**検証環境**  
java version "1.8.0_202"  
Eclipse IDE for Enterprise Java Developers.  
Version: 2019-03 (4.11.0)  
Build id: 20190314-1200  
JUnit4  
mockito-2.23  
powermock-mockito2-2.0.2  
  
# mockitoとpowermock  
Mockitoはテストコードを書く際に利用するモックを提供するライブラリです。  
制限として、コンストラクタやスタティックメソッド、プライベートメソッドをモックできません。  
  
PowerMockはEasyMockとMockitoの両方を拡張し、staticメソッド、finalメソッド、さらにはprivateをモックする機能を備えています。Mockitoと組み合わせて動作することが可能になっています。  
  
今回は以下から「powermock-mockito2-junit-2.0.2.zip」をダウンロードして使用しています。  
https://github.com/powermock/powermock/wiki/Downloads  
  
また、当該環境では下記のJarを別途入手して参照しないと実行時にエラーになりました。  
・byte-buddy-1.9.10.jar  
・byte-buddy-agent-1.9.10.jar  
https://howtodoinjava.com/mockito/plugin-mockmaker-error/  
  
# 使ってみる  
## Mockitoを使う  
### スタブを使う  
下記の例では特定の引数をモックメソッドに渡された場合に固定値を返却するスタブを作成した例になります。  
  
```java
package powermockTest;

import static org.junit.Assert.*;
import static org.mockito.ArgumentMatchers.*;
import static org.mockito.Mockito.*;

import org.junit.Test;
import org.mockito.Mockito;
import org.mockito.invocation.InvocationOnMock;
import org.mockito.stubbing.Answer;

public class Test001 {
	class ClassA {
		public int testInt(int x, int y) {
			return x + y;
		}
	}
	@Test
	public void testWhen1() {
		ClassA mockedA = mock(ClassA.class);
		when(mockedA.testInt(5, 6)).thenReturn(99);
		// モックで指定した結果
		assertEquals(99, mockedA.testInt(5, 6));

		// モック化されていない結果
		assertEquals(0, mockedA.testInt(5, 7));
	}
}
```  
  
#### 引数マッチャー  
モック化する際に指定するパラメータは固定値だけでなく引数マッチャーを使用できます。  
以下の例では固定値のかわりにanyInt()を使用して任意の整数を受け付けるようにできます。  
  
```java
	@Test
	public void testWhen2() {
		ClassA mockedA = mock(ClassA.class);
		// 引数マッチャーを使用して条件を指定可能です。
		// その他の引数マッチャーは以下参照
		// https://javadoc.io/static/org.mockito/mockito-core/3.1.0/org/mockito/ArgumentMatchers.html
		when(mockedA.testInt(anyInt(), anyInt())).thenReturn(99);
		// モックで指定した結果
		assertEquals(99, mockedA.testInt(5, 6));
		assertEquals(99, mockedA.testInt(5, 7));
	}
```  
  
その他以下のような引数マッチャーを使用できます  
https://javadoc.io/static/org.mockito/mockito-core/3.1.0/org/mockito/ArgumentMatchers.html  
  
なお、複数引数があるメソッドの引数の一部だけに対して引数マッチャーを使用することはできません。すべての引数に対して引数マッチャーを使用してください。リテラル値を使いたい場合は以下のような実装になります。  
  
```java
	@Test
	public void testWhen3() {
		// 引数マッチャーを使用した場合、すべての引数はマッチャーによって提供される必要があります。
		/* eq(5)を使わず5を指定すると以下のエラーがでる
		org.mockito.exceptions.misusing.InvalidUseOfMatchersException:
			Invalid use of argument matchers!
			2 matchers expected, 1 recorded:
		*/
		ClassA mockedA = mock(ClassA.class);
		when(mockedA.testInt(eq(5), anyInt())).thenReturn(99);
		// モックで指定した結果
		assertEquals(99, mockedA.testInt(5, 6));
		assertEquals(99, mockedA.testInt(5, 7));
		// モック化されていない結果
		assertEquals(0, mockedA.testInt(6, 7));
	}
```  
  
#### モック化の影響範囲  
jmockitでは@Mockedを使用してモック化されたクラスがある場合、そのテストの期間に作成された全てのインスタンスはモック化されたものとなります。  
mockitoでは次のそのような動作はしません。  
  
```java
	@Test
	public void testWhen4() {
		ClassA mockedA = mock(ClassA.class);
		when(mockedA.testInt(5, 6)).thenReturn(99);

		// モックで指定した結果
		assertEquals(99, mockedA.testInt(5, 6));

		// jmockitと違い、次に作られるインスタンスに影響を与えるわけではない。
		ClassA objA = new ClassA();
		assertEquals(11, objA.testInt(5, 6));

	}
```  
  
#### 同じ引数値で別の結果を返す方法  
同じ引数で別の結果を返すには以下のような実装を行います。  
  
```java
	// 同じ引数で異なる結果を返す方法
	@Test
	public void testWhen5() {
		ClassA mockedA = mock(ClassA.class);
		when(mockedA.testInt(5, 6))
			.thenReturn(99)
			.thenReturn(100);
		// モックで指定した結果
		assertEquals(99, mockedA.testInt(5, 6));
		assertEquals(100, mockedA.testInt(5, 6));
		assertEquals(100, mockedA.testInt(5, 6));

		// 別の書き方
		when(mockedA.testInt(1, 2)).thenReturn(10,20,30);
		assertEquals(10, mockedA.testInt(1, 2));
		assertEquals(20, mockedA.testInt(1, 2));
		assertEquals(30, mockedA.testInt(1, 2));
		assertEquals(30, mockedA.testInt(1, 2));
	}
```  
  
なお、以下のような実装をした場合、最後に記載した内容で上書きされます。  
  
```java
		when(mockedA.testInt(5, 6)).thenReturn(99); // これは無視される
		when(mockedA.testInt(5, 6)).thenReturn(100);
```  
  
#### モック化されたメソッドで例外を出力する方法  
モック化されたメソッドで例外を出力するには以下のような実装を行います。  
  
```java
	@Test
	public void testWhen6() {
		ClassA mockedA = mock(ClassA.class);
		doThrow(new IllegalArgumentException("test")).when(mockedA).testInt(5, 6);
		try {
			// Expectationsで設定した2つめの値が取得
			mockedA.testInt(5, 6);
			fail();

		} catch (IllegalArgumentException ex) {
			assertEquals("test", ex.getMessage());
		}
	}
	@Test
	public void testWhen6_2() {
		ClassA mockedA = mock(ClassA.class);
		when(mockedA.testInt(5, 6)).thenThrow(new IllegalArgumentException("test"));
		try {
			// Expectationsで設定した2つめの値が取得
			mockedA.testInt(5, 6);
			fail();

		} catch (IllegalArgumentException ex) {
			assertEquals("test", ex.getMessage());
		}
	}
```  
  
#### コールバックによるスタブ  
[Answerインターフェイス](https://javadoc.io/static/org.mockito/mockito-core/3.1.0/org/mockito/stubbing/Answer.html)によるスタブを許可されています。  
  
```java
	@Test
	public void testWhen7() {
		ClassA mockedA = mock(ClassA.class);
		when(mockedA.testInt(5, 6)).thenAnswer(
			new Answer<Integer>() {
				public Integer answer(InvocationOnMock invocation) throws Throwable {
		            Object[] args = invocation.getArguments();
		            System.out.println("getArguments----------------");
		            for (Object arg : args) {
			            System.out.println(arg);
		            }
		            // 本物の処理の2倍の値とする
					return (int)invocation.callRealMethod() * 2;
				}
			}
		);

		// モックで指定した結果
		assertEquals(22, mockedA.testInt(5, 6));


	}
```  
  
コールバックメソッドでは[InvocationOnMock](https://javadoc.io/static/org.mockito/mockito-core/3.1.0/org/mockito/invocation/InvocationOnMock.html)が使用可能です。InvocationOnMockを経由して渡された引数やモック化されたメソッドを取得したり、実際のメソッドを実行したりすることが可能です。  
  
#### １ラインでモックを作成する  
下記の実装で1ラインでモックを作成することが可能です。  
  
```java
	@Test
	public void testWhen8() {
		ClassA mockedA = when(mock(ClassA.class).testInt(anyInt(), anyInt())).thenReturn(99).getMock();
		assertEquals(99, mockedA.testInt(5, 6));
	}
```  
  
### Spyを利用した部分モック  
Spyを利用することでオブジェクトの一部のみモック化することが可能です。  
下記のサンプルは特定の引数で実行された場合のみモック化された値を返却します。それ以外は実際のメソッドの実行結果を返却します。  
  
```java
	@Test
	public void testSpy1() {
		ClassA objA = new ClassA();
		ClassA spyA = Mockito.spy(objA);

		when(spyA.testInt(5, 6)).thenReturn(99);

		// モックで指定した結果
		assertEquals(99, spyA.testInt(5, 6));

		// モック化されていない結果
		// mock時とことなり本物がよばれる。
		assertEquals(12, spyA.testInt(6, 6));

	}
```  
  
### Verifyを利用した検証  
verifyを使用することで、モック化されたメソッドがどのように実行されたかを検証することが可能です。  
  
```java
	@Test
	public void testVerify1() {
		ClassA mockedA = mock(ClassA.class);
		mockedA.testInt(5,6);
		mockedA.testInt(3,4);

		// 下記のメソッドが実行されていることを確認する(順番は検証しない)
		verify(mockedA).testInt(3,4);
		verify(mockedA).testInt(5,6);
	}
```  
  
#### 実行された回数の検証  
[times](https://javadoc.io/static/org.mockito/mockito-core/3.1.0/org/mockito/Mockito.html#times-int-)/[never](https://javadoc.io/static/org.mockito/mockito-core/3.1.0/org/mockito/Mockito.html#never--)/[atLeastOnce](https://javadoc.io/static/org.mockito/mockito-core/3.1.0/org/mockito/Mockito.html#atLeastOnce--)/[atMostOnce](https://javadoc.io/static/org.mockito/mockito-core/3.1.0/org/mockito/Mockito.html#atMostOnce--)/[atLeast](https://javadoc.io/static/org.mockito/mockito-core/3.1.0/org/mockito/Mockito.html#atLeast-int-)/[atMost](https://javadoc.io/static/org.mockito/mockito-core/3.1.0/org/mockito/Mockito.html#atMost-int-)等で実行された回数の検証が可能です。  
  
```java
	@Test
	public void testVerify2() {
		ClassA mockedA = mock(ClassA.class);
		mockedA.testInt(5,6);
		mockedA.testInt(3,4);

		// 2回 testInt(?,?)が実行されていること
		verify(mockedA, times(2)).testInt(anyInt(),anyInt());

		// testInt(1,1)は実行されていないことを確認
		verify(mockedA, never()).testInt(1,1);
	}
```  
  
#### 実行順序の検証  
[inOrder](https://javadoc.io/doc/org.mockito/mockito-core/3.1.0/org/mockito/Mockito.html#inOrder-java.lang.Object...-)を用いることでメソッドの実行順番を検証することが可能です。  
  
```java
	class T1 {
		void t1(String a) {

		}
	}
	class T2 {
		void t2(String a) {

		}
	}
	@Test
	public void testVerify3() {
		T1 mocked = mock(T1.class);
		mocked.t1("first");
		mocked.t1("second");

		// 順番を含めて検証する
		InOrder order = inOrder(mocked);
		order.verify(mocked).t1("first");
		order.verify(mocked).t1("second");
	}

	@Test
	public void testVerify4() {
		// 複数のオブジェクトの実行順番を確認することも可能
		T1 t1 = mock(T1.class);
		T2 t2 = mock(T2.class);
		t1.t1("call 1");
		t2.t2("call 2");
		t1.t1("call 3");
		t2.t2("call 4");

		// 順番を含めて検証する
		InOrder order = inOrder(t1, t2);
		order.verify(t1).t1("call 1");
		order.verify(t2).t2("call 2");
		order.verify(t1).t1("call 3");
		order.verify(t2).t2("call 4");
	}
```  
  
### finalのメソッドのモック化  
finalメソッドはデフォルトの設定ではモック化できません。  
finalメソッドのモック化を有効するには下記のファイルを作成する必要があります。  
  
```/mockito-extensions/org.mockito.plugins.MockMaker
mock-maker-inline
```  
  
https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/47856/e3623510-c41b-e932-a8be-2e1bda76425b.png  
  
  
```java
	class ClassB {
		final public int testFinal(int x, int y) {
			return x + y;
		}
	}
	@Test
	public void testFinal1() {
		// finalのメソッドはデフォルトではモック化できない
		// /mockito-extensions/org.mockito.plugins.MockMaker ファイルを作成し、
		// 以下の文字を入れる必要がある
		// mock-maker-inline
		ClassB mockedB = mock(ClassB.class);
		when(mockedB.testFinal(5, 6)).thenReturn(99);

		// モックで指定した結果
		assertEquals(99, mockedB.testFinal(5, 6));

	}
```  
  
## PowerMockを使う  
PowerMockを使用する場合はクラスに下記のアノテーションを付けてください。  
・@RunWith(PowerMockRunner.class)  
・@PrepareForTest({ZZZZ.class, XXX.class})  
  
PrepareForTestにはテスト中にバイトコードレベルで操作する必要があるクラスをしてしてください。staticメソッド、コンストラクタ、プライベートメソッドのモック化を行う場合です。  
  
```java
package powermockTest;

import static org.junit.Assert.*;
import static org.mockito.Mockito.*;

import java.text.SimpleDateFormat;
import java.util.Calendar;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.Mockito;
import org.powermock.api.mockito.PowerMockito;
import org.powermock.core.classloader.annotations.PrepareForTest;
import org.powermock.modules.junit4.PowerMockRunner;

@RunWith(PowerMockRunner.class)
@PrepareForTest({Calendar.class, java.lang.Math.class})
public class Test002 {
}
```  
### Privateメソッドのモック化  
下記の例ではプライベートメソッドのtest1をモック化して、test2経由で実行した場合にモック化された値が返却されたことを確認しています。  
  
```java
	class ClassPrivate {
		private int test1(int x, int y) {
			return x + y;
		}
		public int test2(int x) {
			return test1(x, x) + 1;
		}
	}
	@Test
	public void testPrivate1() {
		ClassPrivate objA = new ClassPrivate();
		ClassPrivate spyA = PowerMockito.spy(objA);
		PowerMockito.when(spyA.test1(5, 5)).thenReturn(99);

		// モックで指定した結果
		assertEquals(100, spyA.test2(5));
		assertEquals(99, spyA.test1(5,5));

	}
```  
  
### Staticメソッドのモック化  
下記の例はjava.lang.Math.randomをモック化した例になります。  
  
```java
	// randomをモック化した例
	@Test
	public void testStatic1() {
		// クラスに以下を設定してください
		// @PrepareForTest({ java.lang.Math.class})
		PowerMockito.mockStatic(java.lang.Math.class);
		PowerMockito.when(java.lang.Math.random()).thenReturn(1.5);

		assertEquals(1.5, java.lang.Math.random(), 0.1);
	}
```  
  
#### 現在時刻の偽装  
Staticメソッドがモック化できると現在時刻を都合のいい時刻に偽装可能です。  
  
```java
	// 現在時刻をモック化した例
	@Test
	public void testStatic2() {
		// クラスに以下を設定してください
		// @PrepareForTest({ Calendar.class})
		Calendar cal = Calendar.getInstance();
		cal.set(2018, 1, 25, 23, 32, 30);
		cal.set(Calendar.MILLISECOND, 0);
		PowerMockito.mockStatic(Calendar.class);
		PowerMockito.when(Calendar.getInstance()).thenReturn(cal);

		Calendar c = Calendar.getInstance();
		SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMddHHmmssSSS");
		assertEquals("20180225233230000", sdf.format(c.getTime()));
	}
```  
  
### メソッド内で作成されたインスタンスをモック化する  
メソッド内で作成されたインスタンスをモック化するにはPowerMockito.whenNewを使用します。  
  
```java
	class classA {
		public String getA() {
			return "getA";
		}
	}
	class classB {
		public String hoge(String x) {
			return x + (new classA()).getA();
		}
	}

	// インスタンスをモックして、テスト対象のメソッド内で使用しているオブジェクトをモック化する
	@Test
	public void testInner() throws Exception {
		classA mockedA = Mockito.mock(classA.class);
		when(mockedA.getA()).thenReturn("abc");

		PowerMockito.whenNew(classA.class).withNoArguments().thenReturn(mockedA);
		classB obj = new classB();
		assertEquals("testabc", obj.hoge("test"));

	}
```  
  
# まとめ  
使用事例が多いのと、学習コストの低さはJMockitよりアドバンテージがあると思います。  
なお、JMockitに存在したカバレッジ計測はできないようです。  
