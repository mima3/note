# 目的  
JavaScriptのフレームワークである[Vue.js](https://vuejs.org/)とCSSのフレームワークである[ bulma](https://bulma.io/)を使用してダイアログを表示してみる。  
  
![image.png](/image/0f9341bd-a959-a184-cecd-e4792f730a83.png)  
  
## サンプル  
Vue.jsにbulmadialog-componentコンポーネントを追加します。  
その後、任意のタイミングで親側からコンポーネントのメソッドshowDialogを実行します。  
この際、以下のプロパティを持つオブジェクトをパラメータに指定します。  
  
 - title : ダイアログのタイトル  
 - contents : ダイアログの内容。HTMLタグは無効  
 - html : ダイアログの内容。HTMLのタグは有効  
 - buttons : ボタン情報の配列  
    - caption : ボタンのタイトル  
    - callback : コールバック関数  
  
### コンポーネント側  
IE11で動くようにしています。  
  
```javascript:bulma_dialog.js
/**
 * bulma + vue.jsでダイアログを表示します。
 * html: vue.jsの管理下に以下を追加します
 *   <bulmadialog-component ref="dialog"></bulmadialog-component>
 * js:Vue.jsを作成するときにコンポーネントを追加する
 *  components: {
      'bulmadialog-component': BulmaDialog,
    },        
 * js:Vue.jsの親のメソッドにて以下を実行
 *  this.$refs.dialog.showDialog({
        title:'わっふるる',
        //contents:'わっふるぼでぃ０\nsadfasfd',
        html : 'あたえたｔ<br>awrawtあたえたｔ<br>',
        buttons : [
          {
            caption : 'はい',
            callback : function () {
              console.log('はい');
            }
          },
          {
            caption : 'いいえ',
            callback : function () {
              console.log('いいえ');
            }
          }
        ]
      });
 */
// eslint-disable-next-line no-unused-vars
const BulmaDialog = {
  /* eslint-disable max-len */
  template: (function() {/*
      <div v-bind:class="{ 'is-active': isShow }" class="modal">
        <div class="modal-background"></div>
          <div class="modal-card">
              <header class="modal-card-head">
                  <p class="modal-card-title">{{data.title}}</p>
              </header>
              <div >
              </div>
              <section v-if="data.html" v-html="data.html" class="modal-card-body"></section>
              <section v-else class="modal-card-body">{{data.contents}}</section>
              <footer class="modal-card-foot"  style="justify-content: flex-end;">
                  <button v-for="btnObj in data.buttons" type="button" class="button" @click="btnObj.callback(); isShow = false;">{{btnObj.caption}}</button>
              </footer>
          </div>
      </div>
      </div>
    */}).toString().match(/\/\*([^]*)\*\//)[1],
  /* eslint-enable */
  data: function() {
    return {
      isShow: false,
      data: {
        title: '',
        body: '',
        html: '',
        buttons: [],
      },
    };
  },
  methods: {
    showDialog: function(data) {
      this.isShow = true;
      this.data.title = data.title;
      this.data.contents = data.contents;
      this.data.html = data.html;
      this.data.buttons = data.buttons;
    },
  },
};

```  
  
### 使う側  
  
```html
<!doctype html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Hello Bulma!</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bulma@0.8.0/css/bulma.min.css">
    <script defer src="https://use.fontawesome.com/releases/v5.3.1/js/all.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <script src="/phptest/codeview/js/bulma_dialog.js"></script>
  </head>
  <body>
  <div id="app">
    <bulmadialog-component ref="dialog"></bulmadialog-component>
    <button class="button is-primary" @click="test1">ダイアログA</button>
    <button class="button is-primary" @click="test2">ダイアログB</button>
  </div>
<script>
var app = new Vue({
  el: '#app',
  components: { //Scopedが使える
    'bulmadialog-component': BulmaDialog,
  },
  data: function() {
      return {
      }
  },
  methods: {
    test1 : function () {
      this.$refs.dialog.showDialog({
        title:'確認（TEXT)', 
        contents: "ふんぐるい むぐるうなふ くとぅるう るるいえ うがふなぐる ふたぐん<br>Ph'nglui mglw'nafh Cthulhu R'lyeh wgah'nagl fhtagn",
        buttons : [
          {
            caption : 'はい',
            callback : function () {
              console.log('test1 はい');
            }
          },
          {
            caption : 'いいえ',
            callback : function () {
              console.log('test2 いいえ');
            }
          }
        ]
      });
    },
    test2 : function () {
      this.$refs.dialog.showDialog({
        title:'確認（HTML)', 
        html: "ふんぐるい むぐるうなふ くとぅるう るるいえ うがふなぐる ふたぐん<br>Ph'nglui mglw'nafh Cthulhu R'lyeh wgah'nagl fhtagn",
        buttons : [
          {
            caption : 'YES',
            callback : function () {
              console.log('Yes');
            }
          },
          {
            caption : 'No',
            callback : function () {
              console.log('No');
            }
          }
        ]
      });
    }
  }
})
</script>

  </body>
</html>
```  
  
# 参考  
**SafariでもエラーにならないJavascriptのヒアドキュメントの書き方**  
https://qiita.com/ampersand/items/c6c773ba7ae9115856d0  
  
# メモ  
Bulma自体がIEのサポートを部分的にのみしかしていないので、テンプレートリテラルを使ってIEをあきらめた方が無難かも  
  
>Bulma uses autoprefixer to make (most) Flexbox features compatible with earlier browser versions. According to Can I use, Bulma is compatible with recent versions of:  
>  
> - Chrome  
> - Edge  
> - Firefox  
> - Opera  
> - Safari  
>  
>Internet Explorer (10+) is only partially supported.  
  
https://github.com/jgthms/bulma  
