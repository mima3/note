昨今、退職エントリーが流行っているので、昨年、勢いで会社を辞めてニートになった記念に何か書こうと思います。  
おっさんは明るい未来に羽ばたくことはできませんでしたので、せめて、ドローンぐらいは明るい未来を羽ばたかせてみせよう、そう思ってこの記事を書いてみました。  
  
# Telloとは  
Telloは小型のドローンでカメラもついており、Android,iPadといった携帯端末で操作が可能です。  
https://www.ryzerobotics.com/jp/tello  
  
この度は無収入のくせに以下のセットを購入しました。  
https://www.amazon.co.jp/gp/product/B07979Q4YS  
  
**注意事項：**  
・充電用のUSBはついてこないので自前でそろえる必要があります。  
![DSC_0032.JPG](https://qiita-image-store.s3.amazonaws.com/0/47856/d2a7bce6-d779-ae59-0120-d6b2b3a2d785.jpeg)  
機体にささないと充電できません。ただし別売りのバッテリーケースを購入すれば機体にささずに充電が可能のようですが、おっさんは無職なので購入してまでの検証はしてません。  
  
・ハードウェアの性能としてはカメラがついているので動画撮影が可能です。つまり、住宅地で飛ばすと覗きとまちがわれるので気をつけましょう。おっさんは無職なのでポリス沙汰になると無職で全国デビューになるので細心の注意しないといけません。  
  
・wifiでつなげて機体の操作をする必要があるため、操作側のリモコンはLANカードが2枚差しでないとインターネットにつなげながらの操作は行えません。おっさんはノートPCを10年ぶりくらいに有線のLANにつなげて作業しました。  
  
・羽に指が当たると、そこそこ痛いので、慣れない間は軍手をして操作したほうがいいです。たぶん、大型のドローンの羽だったら、ドローンのかわりに指が飛んでいたと思います。  
  
・可能なら外の広いところで運転した方が安全です。おっさんは引きこもりなので家でやりましたが、５回ほど壁にあたり墜落しました。  
  
  
# Tello SDK  
TelloはSDKが提供されており、UDP経由で以下のことが行えます。  
・機体の操作。  
・機体の情報取得（傾きとか温度とかバッテリー情報）  
・カメラからの撮影情報の取得  
  
UDPなので基本的に無線LANがつながればどんなプラットフォームでも動作させることができますが、検索してでてくるMacのPythonかC/C++でやった方が絶対にいいです。  
ジャイアントロボのように音声で操作しようと思って、音声認識が簡単にできる.NETで始めたら、えらい苦労しました。  
  
また、SDKではなくて、バイナリデータを送信してSDKに書かれていない操作もできるようですが、ここでは割愛します。  
  
# Tello SDK 1.3.0.0  
以下は[TelloSDK1.3.0.0](https://terra-1-g.djicdn.com/2d4dce68897a46b19fc717f3576b7c6a/Tello%20编程相关/For%20Tello/Tello%20SDK%20Documentation%20EN_1.3_1122.pdf)をそれっぽく翻訳したものです。  
  
  
## 1. 概要  
Tello SDKはWi-Fi UDPポートを介して航空機に接続し、ユーザーはテキストコマンドでドローンを制御することができます。 Tello3.pyファイルをダウンロードするには[ここをクリック](https://terra-1-g.djicdn.com/2d4dce68897a46b19fc717f3576b7c6a/Tello%20编程相关/Both/Tello3(1).py  
)してください。  
  
## 2. アーキテクチャ  
Wi-Fiを使用してTelloとPC、Mac、またはモバイルデバイスとの間の通信を確立します。  
  
### コマンドの送信と応答の受信  
　**Tello IP: 192.168.10.1 UDP PORT:8889 <<-->> PC/Mac/Mobile**  
　**注意１：**同じポートを介してTelloとメッセージを送受信するように、PC、Mac、またはモバイルデバイスでUDPクライアントを設定します。  
  
　**注意２：**他のコマンドを送信する前に、"command"コマンドをUDP ポート8889を介してTelloに送信してTelloのSDKモードを開始します。  
  
### Telloステータスの受信  
　**Tello IP: 192.168.10.1 ->> PC/Mac/Mobile UDP Server: 0.0.0.0 UDP PORT:8890**  
  
　**注意３：**PC、Mac、またはモバイルデバイスにUDPサーバーをセットアップし、UDP PORT 8890を介してIP 0.0.0.0からのメッセージを聞きます。まだ行っていない場合は、注意２を実行して状態データの受信を開始してください。**  
  
### Telloビデオストリームの受信  
　**Tello IP: 192.168.10.1 ->> PC/Mac/Mobile UDP Server:0.0.0.0 UDP PORT:11111**  
　**注意４：**PC、Mac、またはモバイルデバイスにUDPサーバーをセットアップし、UDP PORT 11111を介してIP 0.0.0.0からメッセージを受信します。  
　**注意５：**もし実行していないなら注意２を実行してください。その後、UDP PORT 8889を介して「streamon」コマンドをTelloに送信してストリーミングを開始します。  
  
## 3. TELLOコマンドの種類と結果  
このSDKには3つの基本的なコマンドタイプが含まれています。  
  
**コントロールコマンド(xxx)**  
・コマンドの実行が成功した場合、"ok"が戻ります  
・成功しなかった場合、"error"または有益な結果コードが戻ります  
  
**リードコマンド(xxx?)**  
・サブパラメータの現在値を戻します。  
  
**設定コマンド（xxx a）は、新しいサブパラメータ値を設定しようとします**  
・コマンドの実行が成功した場合、"ok"が戻ります  
・成功しなかった場合、"error"または有益な結果コードが戻ります  
  
## 4. Tello コマンド  
  
### コントロールコマンド  
  
| コマンド       | 説明                 | 考えられる応答               |  
|:---------------|:---------------------|:--------------------------|  
|command         |SDKモードに入る       |ok<br/>error        |  
|takeoff         |Telloが自動で離陸する |ok<br/>error        |  
|land            |Telloが自動で着陸する |ok<br/>error        |  
|streamon        |ビデオストリームをON  |ok<br/>error        |  
|streamoff       |ビデオストリームをOFF |ok<br/>error        |  
|emergency       |全てのモータを停止    |ok<br/>error        |  
|up x            |Tello が x cm上昇<br/>x:20-500 | ok<br/>error        |  
|down x          |Tello が x cm下降<br/>x:20-500 | ok<br/>error        |  
|left x          |Tello が x cm左へ<br/>x:20-500 | ok<br/>error        |  
|right x         |Tello が x cm右へ<br/>x:20-500 | ok<br/>error        |  
|forward x       |Tello が x cm前進<br/>x:20-500 | ok<br/>error        |  
|back x          |Tello が x cm後退<br/>x:20-500 | ok<br/>error        |  
|cw x            |Tello が 時計回りに x度 回転<br/>x:1-3600 | ok<br/>error        |  
|ccw x           |Tello が 反時計回りに x度 回転<br/>x:1-3600 | ok<br/>error      |  
|flip x          |Tello が x 方向に宙返りをする<br/>l:(left)<br/>r(right)<br/>f(foward)<br/>b(back)| ok<br/>error      |  
|go x y z speed  |Tello が x y z の方向へ speed(cm/s)の速度で飛ぶ|x:20-500 *※訳注 前進する xで後退* <br/>y:20-500　*※訳注　左へ -y で右へ*<br/>z:20-500<br/>speed:10-100| ok<br/>error      |  
|curve x1 y1 z1 x2 y2 z2 speed|Telloが現在位置とspeed(cm/s)とともに定義された２つの座標を曲線状に飛びます。もし円弧の半径が0.5-10meterの範囲外の場合、レスポンスはfalseとなります。<br/>x1, x2: 20-500<br/>y1, y2: 20-500<br/>z1, z2: 20-500<br/>speed: 10-60<br/>x/y/z は同時に-20～20の間にはできません|ok<br/>error      |  
  
  
### 設定コマンド  
| コマンド       | 説明                 | 考えられる応答     |  
|:---------------|:---------------------|:-------------------|  
|speed x         |速度 x cm/sを設定する <br/> x: 10-100 |ok<br/>error        |  
|rc a b c d      |４つのチャネルを通してRCコントロールを送信する<br/>a: left/right (-100~100)<br/> b: forward/backward (-100~100)<br/> c: up/down (-100~100) <br/> d: yaw (-100~100) |ok<br/>error        |  
|wifi ssid pass  |Wi-FiのSSIDとpasswordを設定する|ok<br/>error        |  
  
### リードコマンド  
| コマンド       | 説明                 | 考えられる応答     |  
|:---------------|:---------------------|:-------------------|  
|speed?          |現在の速度(cm/s)を取得 |x: 1-100     |  
|battery?        |現在のバッテリーのパーセンテージを取得 |x: 1-100     |  
|time?           |現在の飛行時間を取得|time |  
|height?         |現在の高さ(cm)を取得 |x: 0-3000|  
|temp?           |現在の温度(℃)を取得 |x : 0-90 |  
|attitude?       |IMU(慣性計測装置) の姿勢情報を取得 | pitch roll yaw |  
|baro?           |バロメータ（気圧計）の値(m)を取得| x |  
|acceleration?   |IMU角加速度データを取得する（0.001g）| x y z |  
|tof?            |TOFからの距離(cm)を取得する| x:30-1000|  
|wifi?           |Wi-FiのSNRを取得する| snr|  
  
*訳者注*  
*pitch roll yaw:参考：https://algorithm.joho.info/robotics/roll-pitch-yaw-matrix/*  
*Tof:Time of Flightのことと思われる*  
*SNR:信号対雑音比。SN比が高ければ伝送における雑音の影響が小さく、SN比が小さければ影響が大きい。*  
  
  
## 5. Telloステータス  
**データ型:**String  
**Example:**  
“pitch:%d;roll:%d;yaw:%d;vgx:%d;vgy%d;vgz:%d;templ:%d;temph:%d;tof:%d;h:%d;bat:%d;baro: %.2f; time:%d;agx:%.2f;agy:%.2f;agz:%.2f;\r\n”   
  
**説明**  
o pitch: ピッチ角  
o roll: ロール角  
o yaw: ヨー角  
o vgx: Speed x,  
o vgy: Speed y,  
o vgz: Speed z,  
o templ: 最も低い温度, 摂氏℃  
o temph: 最も高い温度、摂氏℃  
o tof: TOF distance, cm  
o h: Height, cm  
o bat: 現在のバッテリーのパーセンテージ, %  
o baro: バロメーター測定, cm  
o time: モータの時間,  
o agx: 加速度x,  
o agy: 加速度y,  
o agz: 加速度z,   
  
  
## 6 安全機能  
もしTelloが15秒間なにもコマンドを受信しなければ、自動で着陸をします  
  
## 7 TelloのWi-fiリセット  
電源ON状態のTelloに５秒間の長押しをするとインジケータライトが消えて黄色に点滅します。 インジケータランプが黄色のライトを点滅させると、Wi-Fi SSIDとパスワードは工場出荷時の設定にリセットされ、デフォルトではパスワードは設定されません。  
  
  
# .NETで操作してみる。  
おとなしく、Mac＋Pythonで動かした方がいいです。やっている人がいっぱいいます。  
それでもやるなら、以下を参考にしてみてください。  
  
## 事前準備  
・ステータス取得のための8890とビデオストリーム取得のためのポート11111を開けておきます。  
　つながらない場合は、アプリケーション固有のファイアウォールの設定も確認してください。　  
　Telloとのネットワークはパブリックのネットワークになっているはずなので、パブリックの設定もちゃんとみましょう**（２敗）**  
　![settei.png](/image/2637f8e8-63ad-c371-4a1f-95d35ae8aa66.png)  
  
・WireShark等でネットワークの電文をみれるようにしておきます。  
　https://www.wireshark.org/  
　問題の切り分けにやくに立ちます。  
  
・ffmpegを用意する。  
　Telloからのビデオ情報を表示するのに使用します。  
　また、自前でデコードする場合もffmpegのAPIを使用しないと厳しいです。  
　https://www.ffmpeg.org/  
  
  
## 簡単なTelloプログラミング  
画面  
![gamen.png](/image/ca8262fc-8fd7-c0dc-ccce-6e661b947144.png)  
  
```csharp
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;
using System.Net;//for UDP
using System.Net.Sockets; //for UDP
using System.Threading;//for Interlocked
using System.Diagnostics;
using System.IO;
using System.Runtime.InteropServices;

namespace TelloSample
{
    public partial class Form1 : Form
    {
        private UdpClient udpForCmd;     //コマンド結果受信用クライアント
        private UdpClient udpForStsRecv; //ステータスの結果受信用クライアント

        public Form1()
        {
            InitializeComponent();
        }

        // コマンドの結果更新用
        private delegate void DelegateUpdateCmdResult(String ret);

        // ステータスの更新用
        private delegate void DelegateUpdateSts(String sts);

        // コマンド結果を更新。ワーカスレッドからの場合はメインスレッドで実行
        private void UpdateCmdResult(String ret)
        {
            if (this.InvokeRequired)
            {
                Object[] param = new Object[1] { ret };

                this.Invoke(new DelegateUpdateCmdResult(this.UpdateCmdResult), param);
                return;
            }
            this.txtRet.Text = ret;
            this.btnCmd.Enabled = true;
        }

        // ステータスを更新。ワーカスレッドからの場合はメインスレッドで実行
        private void UpdateSts(String sts)
        {
            if (this.InvokeRequired)
            {
                Object[] param = new Object[1] { sts };

                this.Invoke(new DelegateUpdateSts(this.UpdateSts), param);
                return;
            }
            this.txtSts.Text = sts;
        }



        // Telloとの通信を設定する
        private void SetupTello()
        {
            this.udpForCmd = new UdpClient(0);
            this.udpForStsRecv = new UdpClient(8890);
            

            // コマンド結果の受信処理
            Task.Run(() => {
                IPEndPoint remoteEP = null;//任意の送信元からのデータを受信
                while (true)
                {
                    try
                    {
                        String rcvMsg = "";
                        byte[] rcvBytes = udpForCmd.Receive(ref remoteEP);
                        Interlocked.Exchange(ref rcvMsg, Encoding.ASCII.GetString(rcvBytes));
                        this.UpdateCmdResult(rcvMsg);
                    }
                    catch (Exception ex)
                    {
                        Debug.WriteLine(ex.Message);
                    }
                }

            });

            // ステータスの受信処理
            Task.Run(() => {
                IPEndPoint remoteEP = null;//任意の送信元からのデータを受信
                while (true)
                {
                    try
                    {
                        String rcvMsg = "";
                        byte[] rcvBytes = udpForStsRecv.Receive(ref remoteEP);
                        Interlocked.Exchange(ref rcvMsg, Encoding.ASCII.GetString(rcvBytes));
                        rcvMsg = rcvMsg.Replace(";", "\r\n");
                        this.UpdateSts(rcvMsg);
                    }
                    catch (Exception ex)
                    { 
                        Debug.WriteLine(ex.Message);
                    }

                }

            });

        // コマンド送信
        private void sendCmd(string cmd)
        {
            byte[] data = Encoding.ASCII.GetBytes(cmd);
            this.udpForCmd.Send(data, data.Length, "192.168.10.1", 8889);

        }

        // 開始ボタン
        private void btnStart_Click(object sender, EventArgs e)
        {
            SetupTello();

            this.txtRet.Text = "";
            this.btnCmd.Enabled = false;

            sendCmd("command");
        }

        // コマンド送信ボタン押下
        private void btnCmd_Click(object sender, EventArgs e)
        {
            this.txtRet.Text = "";
            this.btnCmd.Enabled = false;
            sendCmd(this.txtCmd.Text);
        }

    }
}

```  
  
### ビデオについて  
streamon コマンドを送信するとポート11111にビデオの情報が受信できます。  
これを表示するにはffmpegのffplayを使用するといいでしょう。  
  
```
ffplay -probesize 32 -sync ext udp://127.0.0.1:11111
```  
  
ウィンドウが起動して現在のカメラが表示されます。  
  
![camera.png](/image/4ab963b0-46c0-4284-026d-00d0fb811f8d.png)  
  
  
### よくある問題  
・コマンドを受け付けない  
無線LANでつながっているかを確認する。  
充電されているか確認する。USBさして青ランプが点灯されたらフル充電である。  
WireSharkでパケットの送受信がされているか確認する。  
送受信のポートが開いているか確認。規定値だとパブリックネットワークなので注意。  
  
・カメラが受信できない。  
11111ポートが開いているか見直す。  
WireSharkでパケットが届いているか確認する。  
  
・ステータスが受信できない。  
8890ポートが開いているか見直す。  
  
・たまにコマンドの応答結果がとれない。  
UDPなので仕様だと思われます。okが必ずくるという前提は多分まずいかもです。  
  
## .NETで自力でビデオデータを取り扱いたい。  
OpenCVSharpを使えば簡単にできます・・・（震え）  
  
・OpenCVSharp  
OpenCvSharp3-AnyCPUとOpenCvSharp4.runtime.winをNuGetで取得していました。  
  
```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using OpenCvSharp;

namespace OpenCVVideo
{
    class Program
    {
        static void Main(string[] args)
        {
            var camera = VideoCapture.FromFile("udp://127.0.0.1:11111");
            using (var normalWindow = new Window("normal"))
            {
                var normalFrame = new Mat();
                var srFrame = new Mat();
                while (true)
                {
                    camera.Read(normalFrame);
                    if (normalFrame.Empty())
                        break;

                    normalWindow.ShowImage(normalFrame);
                    int key = Cv2.WaitKey(100);
                    if (key == 27) break;   // ESC キーで閉じる
                }
            }

        }
    }
}

````  
  
OpenCVのVideoCapture.FromFileはファイルといいつつ、UDPからのストリームもとれます。  
まちがっても自分でUDPで11111ポートを監視してデコードしようとしてはいけません。  
以下にその愚かな例をしめしますが、おとなしくOpenCVを使いましょう。  
  
### 愚かにも自前でUDPの11111ポートを監視した例  
簡単な流れとしては以下の通りになります。  
  
1. streamonコマンドを送信  
2. ポート11111を監視  
3. 1460バイトとどいている間は後続のパケットがあるので受信しつづける。1460以外のデータ長がきたら、いままできたぶんとまとめてffmpegのAPIを使用してデコードする。  
4. デコード結果をffmpegの関数を使用してRGBに変換する  
5. OpenCVにかませるため、OpenCVの関数をつかってBGRに変換する  
6. OpenCVでビデオつくったりする。  
  
この挙動は下記を参考にしました。  
https://github.com/dji-sdk/Tello-Python/blob/master/Tello_Video/tello.py  
https://github.com/dji-sdk/Tello-Python/tree/master/Tello_Video/h264decoder  
  
.NETでつらいのは3と4です。  
これをおこなうにはネイティブのDLLでffmpegのAPIを使い実行した結果をC#に渡す必要があります。  
  
#### .NETでTelloのビデオを扱うために必要なライブラリ  
・ffmpeg  
https://ffmpeg.zeranoe.com/builds/  
devにincludeファイルとlibファイル、sharedにdllがあるのでそれぞれダウンロードしました。  
このライブラリはtelloから取得したh264形式をRGBに変換するために使用します。  
ソースコードから自前でコンパイルもできますが、MSYS2をいれたりして結構、手間がかかるのでFormアプリでいいなら、おとなしく配布されているものを使った方がいいと思います。（一敗）  
  
  
#### h264decorderの移植  
先に紹介したPythonでh264のデコードをするためのコードを.NETでやるために改造しました。  
https://github.com/mima3/Tello/tree/master/h264decoder  
  
おそらく、メモリ解放処理がうまくできていない気がするので参考程度にしてください。  
  
  
#### h264decoderの使用  
.NETでネイティブのDLLを使用する場合は、32bitか64bitかは意識してください。  
今回は64bitで動かすためにAnyCPUをx64に変更するか、32bitを優先するフラグをオフにする必要があります。  
もし、今まで動いていたアプリケーションが起動すらしない場合、DLLが32bitと64bitで混在している可能性を疑ってみてください。  
  
以下に移植したh264decoderを使用して動画を作成する実装例を記載します。  
  
```csharp
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;
using System.Net;//for UDP
using System.Net.Sockets; //for UDP
using System.Threading;//for Interlocked
using System.Diagnostics;
using OpenCvSharp;
using System.IO;
using System.Runtime.InteropServices;

namespace TelloSample
{
    public partial class Form1 : Form
    {
        [StructLayout(LayoutKind.Sequential, CharSet = CharSet.Ansi)]
        public struct H264DecoderResult
        {
            public int w;
            public int h;
            public int size;
            public IntPtr buff;
        }
        [DllImport("h264decoder.dll", EntryPoint = "InitH264Decoder")]
        static extern void _InitH264Decoder();

        [DllImport("h264decoder.dll", EntryPoint = "TermH264Decoder")]
        static extern void _TermH264Decoder();

        [DllImport("h264decoder.dll", EntryPoint = "DecodeH264")]
        static extern bool _DecodeH264(IntPtr buff, int size, ref H264DecoderResult outbuff);

        [DllImport("h264decoder.dll", EntryPoint = "GetH264DecoderLastError")]
        public static extern IntPtr GetH264DecoderLastError();

        [DllImport("h264decoder.dll", EntryPoint = "FreeData")]
        public static extern void FreeData(IntPtr data);

        [DllImport("kernel32.dll", EntryPoint = "CopyMemory", SetLastError = false)]
        public static extern void CopyMemory(IntPtr dest, IntPtr src, uint count);


        private UdpClient udpForCmd;     //コマンド結果受信用クライアント
        private UdpClient udpForStsRecv; //ステータスの結果受信用クライアント
        private UdpClient udpForVideo;   //ビデオストリームの受信用

        public Form1()
        {
            InitializeComponent();
        }

        // コマンドの結果更新用
        private delegate void DelegateUpdateCmdResult(String ret);

        // ステータスの更新用
        private delegate void DelegateUpdateSts(String sts);

        // コマンド結果を更新。ワーカスレッドからの場合はメインスレッドで実行
        private void UpdateCmdResult(String ret)
        {
            if (this.InvokeRequired)
            {
                Object[] param = new Object[1] { ret };

                this.Invoke(new DelegateUpdateCmdResult(this.UpdateCmdResult), param);
                return;
            }
            this.txtRet.Text = ret;
            this.btnCmd.Enabled = true;
        }

        // ステータスを更新。ワーカスレッドからの場合はメインスレッドで実行
        private void UpdateSts(String sts)
        {
            if (this.InvokeRequired)
            {
                Object[] param = new Object[1] { sts };

                this.Invoke(new DelegateUpdateSts(this.UpdateSts), param);
                return;
            }
            this.txtSts.Text = sts;
        }



        // Telloとの通信を設定する
        private void SetupTello()
        {
            this.udpForCmd = new UdpClient(0);
            this.udpForStsRecv = new UdpClient(8890);
            this.udpForVideo = new UdpClient(11111);


            // コマンド結果の受信処理
            Task.Run(() => {
                IPEndPoint remoteEP = null;//任意の送信元からのデータを受信
                while (true)
                {
                    try
                    {
                        String rcvMsg = "";
                        byte[] rcvBytes = udpForCmd.Receive(ref remoteEP);
                        Interlocked.Exchange(ref rcvMsg, Encoding.ASCII.GetString(rcvBytes));
                        this.UpdateCmdResult(rcvMsg);
                    }
                    catch (Exception ex)
                    {
                        Debug.WriteLine(ex.Message);
                    }
                }

            });

            // ステータスの受信処理
            Task.Run(() => {
                IPEndPoint remoteEP = null;//任意の送信元からのデータを受信
                while (true)
                {
                    try
                    {
                        String rcvMsg = "";
                        byte[] rcvBytes = udpForStsRecv.Receive(ref remoteEP);
                        Interlocked.Exchange(ref rcvMsg, Encoding.ASCII.GetString(rcvBytes));
                        rcvMsg = rcvMsg.Replace(";", "\r\n");
                        this.UpdateSts(rcvMsg);
                    }
                    catch (Exception ex)
                    { 
                        Debug.WriteLine(ex.Message);
                    }

                }

            });


            // ビデオストリームの受信処理
            Task.Run(() => {
                IPEndPoint remoteEP = null;//任意の送信元からのデータを受信
                byte[] packetData = new byte[0];
                int cnt = 0;
                _InitH264Decoder();
                var fourcc = VideoWriter.FourCC('m', 'p', '4', 'v');
                var video = new VideoWriter("test.mp4", fourcc, 20, new OpenCvSharp.Size(960, 720) );

                while (true)
                {
                    try
                    {

                        byte[] rcvBytes = udpForVideo.Receive(ref remoteEP);
                        int l = packetData.Length;
                        Array.Resize<byte>(ref packetData, l + rcvBytes.Length);
                        Array.Copy(rcvBytes, 0, packetData, l, rcvBytes.Length);
                        if (rcvBytes.Length != 1460)
                        {

                            int size = Marshal.SizeOf(packetData[0]) * packetData.Length;
                            IntPtr inPtr = Marshal.AllocHGlobal(size);
                            Marshal.Copy(packetData, 0, inPtr, packetData.Length);

                            H264DecoderResult decret = new H264DecoderResult();
                            Debug.WriteLine("DO DECODE");
                            if (_DecodeH264(inPtr, packetData.Length, ref decret))
                            {
                                Debug.WriteLine("DO DECODE,,,,ok");
                                var mat = new Mat(decret.h, decret.w, MatType.CV_8UC3);
                                CopyMemory(mat.Data, decret.buff, (uint)decret.size);
                                var matCv = new Mat();
                                Cv2.CvtColor(mat, matCv, ColorConversionCodes.RGB2BGR);
                                video.Write(matCv);
                                FreeData(decret.buff);
                            }
                            else
                            {
                                Debug.Write(Marshal.PtrToStringAnsi(GetH264DecoderLastError()));
                            }
                            Marshal.FreeHGlobal(inPtr);


                            packetData = new byte[0];
                            ++cnt;

                        }
                    }
                    catch (Exception ex)
                    {
                        Debug.WriteLine(ex.Message);
                    }

                }
                // 後片付けの方法はあとで考える。（呼ばれない）
                _TermH264Decoder();
                video.Release();
            });

        }

        // コマンド送信
        private void sendCmd(string cmd)
        {
            byte[] data = Encoding.ASCII.GetBytes(cmd);
            this.udpForCmd.Send(data, data.Length, "192.168.10.1", 8889);

        }

        // 開始ボタン
        private void btnStart_Click(object sender, EventArgs e)
        {
            SetupTello();

            this.txtRet.Text = "";
            this.btnCmd.Enabled = false;

            sendCmd("command");
        }

        // コマンド送信ボタン押下
        private void btnCmd_Click(object sender, EventArgs e)
        {
            this.txtRet.Text = "";
            this.btnCmd.Enabled = false;
            sendCmd(this.txtCmd.Text);
        }

    }
}

```  
  
とりあえず64bitで動くソースは以下に置いておきます。  
https://github.com/mima3/Tello  
  
メモリ解放関係がだいぶ怪しいので、とりあえず動かす用としてくださいというか、そもそもOpenCVでやった方がはるかに楽です・・・orz  
  
  
# 最後に  
ここではTelloの最低限の機能を.NETで実装した例をしめしました。  
基本的に文字のコマンドを送信するだけで、制御できますが、h264のデコード処理はネィティブのDLLを作ってOpenCVがとれるようにする必要があります。  
そこを超えてしまえば、あとは.NETのライブラリを色々と利用してにTelloを活用する道筋が見えるかと思います。（たとえば、音声認識で飛行させるとか・・・）  
  
~~まぁ、.NETでTelloを使う道筋はみえても、おっさんの人生の道筋はみえないね。しかたないね。~~  
