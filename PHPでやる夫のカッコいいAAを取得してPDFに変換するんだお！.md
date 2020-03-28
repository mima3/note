![yaruo2.png](/image/329e8a2a-fb75-82cd-f60e-865b1a380aa5.png)  
  
  
 **やるおスレをPDF化するお！**  
http://needtec.sakura.ne.jp/AaPdfConverter/  
  
ということで、やる夫板のスレッドからAAを取得して、それをPDFに変換してダウンロードする方法について説明します。  
  
## read.cgiが指定されたらrawmode.cgiに置き換える。  
したらばでは 「read.cgi」よりサーバの側処理コストが低く、フォーマットも単純な「rawmode.cgi」があるので、そっちにアクセスしましょう。  
http://blog.livedoor.jp/bbsnews/archives/50283526.html  
  
フォーマットが単純すぎて、simplehtmldomを使わなくて済みます。  
  
もし、rawmode.cgiが存在しない場合は以下の手順で、HTMLを解析する必要があります。  
  
### PHPでのスクレイピング（現在未使用）  
PHPでスクレイピングするためにsimple_html_domを使用します。  
下記のページからダウンロードして任意の場所に配置してください。  
  
 **simple_html_dom**   
http://simplehtmldom.sourceforge.net/  
  
2014/12/27時点での最新は simplehtmldom_1_5となります。  
simple_html_domはfile_get_html()関数でHTTP経由で直接パースすることができますが、これは避けた方がいいです。  
  
  
まず、「Curl」でダウンロードしてからsimplehtmldomを使用した時より遅くなります。  
  
参考：  
 **PHPのsimple_html_domでスクレイピングするとき「file_get_html」ではなく「Curl」を使ったら取得実行時間が半分（2倍速）になった**  
http://ccby.hatenablog.com/entry/2014/10/20/190645  
  
そして、ある一定の量のページを取得するときに空文字が返ります。当然、どのようなエラーが発生したかわかりません。  
  
以上の２点の理由により、上記のページを参考にCurlで取得した文字列をsimplehtmldomで解析したほうが良いでしょう。  
  
  
  
## PHPでPDFの出力  
PHPでPDFを出力するにはtcpdfを用いるといいでしょう。  
  
http://www.tcpdf.org/  
  
2014/12/27時点での最新は  tcpdf_6_2_3.zip となります。  
  
また、AA出力に適したモナ―フォントで出力するには下記の手順が必要です。  
  
１．モナーフォントを下記のページから取得します。  
http://www.geocities.jp/ipa_mona/  
  
展開したファイルのfonts/ipagp-mona.ttfを使用します。  
  
２．ipagp-mona.ttfをtcpdfで使用できる形式に変換します。  
tcpdf/toolsの以下のコマンドを実行します。  
  
```
php C:\pleiades-e3.6\xampp\htdocs\AaPdfConverter\tcpdf\tools\tcpdf_addfont.php -i ipagp-mona.ttf
```  
  
これによりtcpdf/fontsに以下のファイルが作成されます。  
・ipagpmona.ctg.z  
・ipagpmona.php  
・ipagpmona.z  
  
３．以降次のようなコードでフォントが指定できるようになります。  
  
```php
$pdf = new TCPDF(PDF_PAGE_ORIENTATION, PDF_UNIT, PDF_PAGE_FORMAT, true, 'UTF-8', false);
$pdf->SetFont('ipagpmona', '', 8);
```  
  
## AA判定のアルゴリズム  
書き込みにAAが含まれているかどうかの判定は以下のサイトを参考にします。  
  
 **AA（アスキーアート）簡易判定アルゴリズム**  
http://d.hatena.ne.jp/awef/20110412/1302605740  
  
## PDF変換のコード  
  
```php:create_pdf.php
<?php
if(!isset($_GET['target']))
{
  echo "No URL data.";
  exit();
}

// 10分前のものは消す
$rmtime = time() - 60*10;
$dir_h = opendir( "./tmp" ) ;
while (false !== ($file = readdir($dir_h))) 
{
  if( filemtime( "./tmp/".$file) < $rmtime )
  {
    @unlink( "./tmp/".$file);
  }
}
closedir( $dir_h ) ;

session_regenerate_id(true);
session_start();
set_time_limit(0);
require_once('tcpdf/tcpdf.php');


function dlPage($href) {
    $curl = curl_init();
    curl_setopt($curl, CURLOPT_SSL_VERIFYPEER, FALSE);
    curl_setopt($curl, CURLOPT_HEADER, false);
    curl_setopt($curl, CURLOPT_FOLLOWLOCATION, true);
    curl_setopt($curl, CURLOPT_URL, $href);
    curl_setopt($curl, CURLOPT_REFERER, $href);
    curl_setopt($curl, CURLOPT_RETURNTRANSFER, TRUE);
    curl_setopt($curl, CURLOPT_USERAGENT, "Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US) AppleWebKit/533.4 (KHTML, like Gecko) Chrome/5.0.375.125 Safari/533.4");
    $str = curl_exec($curl);
    curl_close($curl);
    return $str;
}

$url = $_GET['target'];
$url = htmlspecialchars($url);
if (strstr($url, '/read.cgi/')) {
    $url = str_replace('/read.cgi/', '/rawmode.cgi/', $url);
}

if( !$html = dlPage($url) )
{
  echo 'ファイルが読み込めません';
  exit();
}

$pdf = new TCPDF(PDF_PAGE_ORIENTATION, PDF_UNIT, PDF_PAGE_FORMAT, true, 'UTF-8', false);
$pdf->setFontSubsetting(false);  // フォントの組み込み
$pdf->SetFont('ipagpmona', '', 8);
$contents = mb_convert_encoding($html, 'utf-8', 'euc-jp');
$lines = preg_split("(<>)", $contents);
foreach ($lines as $text) {
  if( strpos($text,'　 ') !== false )
  {
    $pdf->AddPage('P', 'A4');
    $text=str_replace('<br>',"\n",$text);
    $text=str_replace('&quot','"',$text);
    $text=str_replace('&lt','<',$text);
    $text=str_replace('&gt','>',$text);
    $pdf->MultiCell(0,10,$text);
  }
}
$tempfile = './tmp/'. session_id(). '.pdf';
touch($tempfile);
$real_tempfile = realpath($tempfile); // 実際にファイルがないと作れない
$pdf->Output($real_tempfile, 'F');    // 絶対パスである必要がある。

session_destroy();
header("HTTP/1.1 301 Moved Permanently");
header("Location: " . $tempfile );
?>

```  
  
以上の手順により、アスキーアートを含む書き込みをPDFに変換することが可能になります。  
  
### Chromeで表示されない場合  
フォント全体を埋め込んどかないとダメっぽいです。  
  
```
$pdf->setFontSubsetting(false);  // フォントの組み込み
```  
  
https://code.google.com/p/chromium/issues/detail?id=465322  
