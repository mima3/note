# 目的  
WordPressでPlantUMLを利用して図を作成します。  
なおこの際、日ノ本言葉を使うものとします。  
![plantumlphp.png](/image/36695a56-04bc-b068-baf3-d86a01bcbd1e.png)  
  
# 環境  
PHP　7.3  
Wordpress　5.2  
  
# PlantUML Renderer  
PlantUML RendererはWordPressのプラグインとして以下に公開されています。  
https://ja.wordpress.org/plugins/plantuml-renderer/  
  
WordPressのソースブロックに以下を記載します。  
  
```text
[plantuml]
Alice -> Bob: Authentication Request
Bob --> Alice: Authentication Response

Alice -> Bob: Another authentication Request
Alice <-- Bob: another authentication Response
[/plantuml]
```  
  
以下のような画像が表示されます。  
  
![plantumlphp0.png](/image/753c0904-9c09-d18f-d620-570cc0d58f3b.png)  
  
  
**ただし当然の権利のように日ノ本言葉を記載すると文字化けします。**  
  
## 仕組み  
PlantUML Renderのソースは以下にあります。  
https://plugins.trac.wordpress.org/browser/plantuml-renderer/trunk/public/class-plantuml-renderer-public.php  
  
```php
	        public function pumlr_shortcode_handler( $atts, $content = null ) {
	                $new_content = str_replace( '&gt;','>',$content );                                      // Right angle bracket.
	                $new_content = str_replace( '&lt;','<',$new_content );                  // Left angle bracket.
	                $new_content = str_replace( '&#8211;','--',$new_content );      // En-dash  (WP converts "--" to one en-dash).
	                $new_content = str_replace( '&#8212;','--',$new_content );      // Em-dash  (WP converts "--" to one em-dash).
	                $new_content = str_replace( '&#8220;','"',$new_content );         // Curly left double quote
	                $new_content = str_replace( '&#8221;','"',$new_content );         // Curly right double quote
	                $new_content = str_replace( '&#8221;','"',$new_content );         // Curly right double quote
	                $remove_array = array( '<p>', '</p>', '<br />', '<br/>' );              // Remove HTML tags.
	                $new_content = str_replace( $remove_array, '', $new_content );
	                $img = '<p><img src=http://www.plantuml.com/plantuml/img/';
	                $img .= $this->encodep( $new_content );
	                $img .= ' alt="PlantUML Syntax:' . $content . '" usemap="#plantuml_map">';
	
	                // Get the image map, if our syntax has plantuml link syntax.
	                $linkpos = strpos( $new_content, '[[' );
	                if ( false !== $linkpos ) {
	                        $response = wp_remote_get( 'http://www.plantuml.com/plantuml/map/' . $this->encodep( $new_content ) );
	                        if ( is_array( $response ) ) {
	                                $body = $response['body'];
	                                $img .= $body;
	                        }
	                }
	                $img .= '</p>';
	                return $img;
	        }
	        /**
	         * Encode our plantuml syntax - See PlantUML PHP Doc for details - where this was lifted from.
	         *
	         * @param    string $text Our text to encode.
	         * @since    1.0.0
	         */
	        private function encodep( $text ) {
	                $data = utf8_encode( $text );
	                $compressed = gzdeflate( $data, 9 );
	                return $this->encode64( $compressed );
	        }
```  
  
1. UTF-8でエンコードする  
2. 圧縮する  
3. base64で表現する  
4. 「http://www.plantuml.com/plantuml/map/3の結果」を呼び出す。  
5. 画像が作成されるのでimgタグで表示する  
  
この仕組みは以下のページに記載されており、プラグインはそれを忠実に実装しています。  
http://plantuml.com/ja/code-php  
  
上記ページからたどれるPHPのサンプルでは同様に化けます。  
  
## JavaScriptでの実装  
PlantUMLの公式ページでPHPではなく、JavaScriptで同じことをしている箇所があります。  
http://plantuml.com/en/demo-javascript-synchronous  
  
このページでは日本語が化けることはありません。  
どんな実装をしているか見てみましょう。  
  
```javascript:demo-javascript-synchronous
function compress(a){
  a=unescape(encodeURIComponent(a));
  GID("im").src="http://www.plantuml.com/plantuml/img/"+encode64(zip_deflate(a,9))
};
```  
  
encode64とzip_deflateはsynchro.jsに実装されていてPHP側の実装と同様です。  
問題はこのパラメータに与えるaにあります。  
  
ここではaをencodeURIComponentをしたあと、unescapeを実行しています。  
unescapeはASCII以外対応していないので日本語は文字化けします。  
そして、その化けた内容をそのまま利用しています。  
  
つまり、PHPのutf8_encodeが不要ということになります。  
  
## 修正内容  
インストールしたプラグインのファイルを直接修正して対応します。  
このファイルは以下に存在してました。  
/ワードプレスのパス/wp-content/plugins/plantuml-renderer/public  
  
  
utf8_encodeを除去します  
  
```php:class-plantuml-renderer-public.php
	/**
	 * Encode our plantuml syntax - See PlantUML PHP Doc for details - where this was lifted from.
	 *
	 * @param    string $text Our text to encode.
	 * @since    1.0.0
	 */
	private function encodep( $text ) {
		//$data = utf8_encode( $text );
		$data = $text; // http://plantuml.com/en/demo-javascript-synchronous のJSではunescape(encodeURIComponent(a))でASCII以外は化けているデータを送ってうまくいっている
		$compressed = gzdeflate( $data, 9 );
		return $this->encode64( $compressed );
	}

```  
  
また、「"」が混ざった場合、imgタグのaltが上手く表示できなくなるのでそれもあわせてなおします。ここでは全角に変換してます。  
  
```php:class-plantuml-renderer-public.php
	/**
	 * Our lovely shortcode.
	 *
	 * @param    array  $atts    Attributes passed into shortcode.
	 * @param    string $content Content within the shortcode tags.
	 * @since    0.0.3
	 */
	public function pumlr_shortcode_handler( $atts, $content = null ) {
		$new_content = str_replace( '&gt;','>',$content );					// Right angle bracket.
		$new_content = str_replace( '&lt;','<',$new_content );			// Left angle bracket.
		$new_content = str_replace( '&#8211;','--',$new_content );	// En-dash  (WP converts "--" to one en-dash).
		$new_content = str_replace( '&#8212;','--',$new_content );	// Em-dash  (WP converts "--" to one em-dash).
		$new_content = str_replace( '&#8220;','"',$new_content );	  // Curly left double quote
		$new_content = str_replace( '&#8221;','"',$new_content );	  // Curly right double quote
		$new_content = str_replace( '&#8221;','"',$new_content );	  // Curly right double quote
		$remove_array = array( '<p>', '</p>', '<br />', '<br/>' );		// Remove HTML tags.
		$new_content = str_replace( $remove_array, '', $new_content );
		$img = '<p><img src=http://www.plantuml.com/plantuml/img/';
		$img .= $this->encodep( $new_content );
		// "を全角に変換して表示する
		$img .= ' alt="PlantUML Syntax:' . str_replace( '"', '”', $content) . '" usemap="#plantuml_map">';

		// Get the image map, if our syntax has plantuml link syntax.
		$linkpos = strpos( $new_content, '[[' );
		if ( false !== $linkpos ) {
			$response = wp_remote_get( 'http://www.plantuml.com/plantuml/map/' . $this->encodep( $new_content ) );
			if ( is_array( $response ) ) {
				$body = $response['body'];
				$img .= $body;
			}
		}
		$img .= '</p>';
		return $img;
	}

```  
  
## 結果  
上記２つの対応により、日ノ本言葉がつかえるようになります。  
![plantumlphp.png](/image/36695a56-04bc-b068-baf3-d86a01bcbd1e.png)  
  
  
## 改造案  
いまのままだと外部のURLに頼ってしまうので自前のサーバにPlantUMLのJARを配置した方がいいでしょう。  
以下のページが参考になりそうです。  
http://dora.bk.tsukuba.ac.jp/~takeuchi/?%E3%82%BD%E3%83%95%E3%83%88%E3%82%A6%E3%82%A7%E3%82%A2%2Fpukiwiki%2Fuml.inc.php  
