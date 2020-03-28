# はじめに  
データベースの場合は、挿入、更新、削除をやったあとで取り消したければロールバックできます。  
地上の哀れな羊は、データベースと同様にファイルにおいてもコピー、移動、削除、変更を行ったあとで、やっぱり取り消しが行えるよう神たるマイクロソフトに祈りました。  
  
偉大なるマイクロソフトはVista開発時に、[Transactional NTFS(TxF)](https://docs.microsoft.com/ja-jp/windows/win32/fileio/transactional-ntfs-portal)を遣わし、哀れな羊を救いを与えました。  
  
しかし、この救いはWindows8の頃から非推奨となり、「[Transaction NTFSの代替を考えておくよう](https://docs.microsoft.com/ja-jp/windows/win32/fileio/deprecation-of-txf)」、ファイルだけでなく自分自身をロールバックするというアメリカンジョークをなさいました。  
  
神に見捨てられた哀れな羊たちはStackOverflowに集い、代替方法を考えることになりました。  
  
**Alternatives to using Transactional NTFS**  
https://stackoverflow.com/questions/13420643/alternatives-to-using-transactional-ntfs  
  
この際、NuGetより救いの手を差し伸べたのが.NET Transactional File Managerなのです。  
  
**TransactionalFileMgr**　.NET Transactional File Manager  
https://archive.codeplex.com/?p=transactionalfilemgr  
  
# インストール方法  
NuGetでTxFileMangerをインストールしてください。  
![image.png](/image/401bca73-ab44-10fd-6611-971531723677.png)  
  
1.3と2.0があります。2.0は2019年の4月ころに追加されたもので、いくつかの機能が追加されているようですが、今回は1.3を使用することにします。  
  
# サンプル  
  
```csharp
using ChinhDo.Transactions;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Transactions; // System.Transactions.dll

namespace FileTest
{
    class Program
    {
        static void Main(string[] args)
        {
            TxFileManager fileMgr = new TxFileManager();
            using (TransactionScope scope1 = new TransactionScope())
            {
                try
                {
                    // 「newfolder」ディレクトリを作成
                    fileMgr.CreateDirectory(@"test\a\newfolder");

                    // 「newfolder\aaa.txt」ファイルを作成
                    fileMgr.WriteAllText(@"test\a\newfolder\aaa.txt", "あああああ");

                    // 「newfolder\aaa.txt」ファイルに追記
                    fileMgr.AppendAllText(@"test\a\newfolder\aaa.txt", "いいいい");

                    // test.txt→test_renamed.txt に変更
                    fileMgr.Move(@"test\a\test.txt", @"test\a\test_renamed.txt");

                    // WriteAllTextはBOMなしのUTF8になるので、エンコードを変えたかったりバイナリを操作したい場合はSnapshotを行う
                    fileMgr.Snapshot(@"test\a\test_renamed.txt");
                    System.IO.StreamWriter sw = new System.IO.StreamWriter(@"test\a\test_renamed.txt", false, System.Text.Encoding.Unicode);
                    sw.Write("ああああああ");
                    sw.Close();

                    String ret;
                    Console.WriteLine("Completeを実行する場合は[Y]を入力してください");
                    ret = Console.ReadLine();
                    if (ret.Equals("Y"))
                    {
                        scope1.Complete();
                        Console.WriteLine("ファイル操作完了");
                    }
                    else
                    {
                        Console.WriteLine("ロールバック");
                    }

                }
                catch (Exception ex)
                {
                    Console.WriteLine(ex);
                }
            }
            Console.ReadLine();
        }
    }
}

```  
  
## サンプルの実行結果  
(1)実行前  
![image.png](/image/58aa4f4e-f834-8bdb-6ca2-21a3e4f6da5e.png)  
  
(2)Complete実行前  
![image.png](/image/5aaaea58-1e5a-9dbf-3bd5-45a6f1ca7718.png)  
  
(3-1)Completeした場合  
![image.png](/image/5aaaea58-1e5a-9dbf-3bd5-45a6f1ca7718.png)  
・・・(2)より変化なし  
  
(3-2)Completeしない場合  
![image.png](/image/58aa4f4e-f834-8bdb-6ca2-21a3e4f6da5e.png)  
・・・(1)と同じになる  
  
## サンプルの解説  
・TransactionScope 内のTxFileManager経由のファイル操作についてトランザクション処理を行います。  
  
・scope1.Complete()を行った場合は、TxFileManager経由で行ったファイル操作は、そのままになります。  
・Complete()を行わないでスコープを抜けた場合は元に戻ります。  
**※ただし、バックアップから地道に差し戻しているだけなので、ファイルがロックされていたり、アプリケーション自体が強制終了された場合は元に戻りません**  
  
# サポートしている機能について  
説明の詳細についてはTxFileManager.csと各操作の～Operation.csを見て記載しています。  
  
## AppendAllText(string path, string contents)  
指定のファイルに追記を行う。  
  
トランザクションの外では「File.AppendAllText」を実行するのみ  
  
トランザクション内では操作対象のファイルが存在する場合はバックアップとしてコピーしてから「File.AppendAllText」を実行する。  
ロールバック時にバックアップから復元される。仮に今回のトランザクションで新規で作成したファイルだった場合は、ロールバック時に削除される。  
  
## Copy(string sourceFileName, string destFileName, bool overwrite)  
コピー元のパスからコピー先のパスにファイルのコピーを行う  
  
トランザクションの外では「File.Copy」を実行するのみ  
  
トランザクション内ではコピー先のファイルが存在する場合はバックアップとしてコピーしてから「File.Copy」を実行する。  
ロールバック時にバックアップから復元される。  
  
## CreateDirectory(string path)  
ディレクトリの作成を行う。  
  
トランザクション外では「Directory.CreateDirectory」を実行するのみ  
トランザクション内ではすでに存在しているディレクトリか記憶してから「Directory.CreateDirectory」を実行する。  
ロールバック時はすでに存在していた場合はなにもしないが、存在しておらずトランザクションで作成されたディレクトリの場合は削除をする。  
  
## DeleteDirectory(string path)  
ディレクトリの削除を行う。  
  
トランザクション外では「Directory.Delete」を実行するのみ。  
トランザクション内では削除対象のディレクトリをバックアップに移動する。  
ロールバック時には、バックアップから、移動したディレクトリを元の位置に戻す。  
  
## Delete(string path)  
ファイルの削除を行う。  
  
トランザクション外では「File.Delete(path)」を実行する。  
トランザクション内ではバックアップに削除対象のファイルをコピーしてから「File.Delete(path)」を実行する。  
ロールバック時にバックアップから復元する。  
  
## Move(string srcFileName, string destFileName)  
ファイルの移動を行う。  
トランザクション外では「File.Move」を実行する。  
トランザクション内では移動元と移動先を記憶してから「File.Move」を実行する。  
ロールバック時には移動先から移動元にファイルを移動することで元に戻す。  
  
## Snapshot(string fileName)  
ファイルのスナップショットを保存しておく。  
これにより、ロールバック時にファイルを元に戻すことができる。  
  
トランザクション外では何もしない  
  
## WriteAllText(string path, string contents)  
指定のファイルの作成または上書きをする。  
  
トランザクションの外では「File.WriteAllText」を実行するのみ  
  
トランザクション内では操作対象のファイルが存在する場合はバックアップとしてコピーしてから「File.WriteAllText」を実行する。  
ロールバック時にバックアップから復元される。仮に今回のトランザクションで新規で作成したファイルだった場合は、ロールバック時に削除される。  
  
  
# まとめ  
アプリケーションの強制終了や、別プロセスの操作などを考えた場合、完璧ではないとはいえませんが、簡単なファイル操作であればTransactionalFileMgrでロールバックが行えることがわかりました。  
  
また、ソースコードがダウンロードできるので、機能拡張もわりと容易に行えるかと思います。  
