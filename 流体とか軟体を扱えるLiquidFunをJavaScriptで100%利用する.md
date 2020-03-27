LiquidFunは流体とか軟体を扱うためのライブラリで、Box2dをベースに実装しています。  
基本的にC++で実装されていますが、JavaまたはJavaScriptからも使用できます。  
今回はJavaScriptでLiquidFunを利用する方法について説明します。  
  
  
 **Liquidで表現したゆっくり**   
https://qiita-image-store.s3.amazonaws.com/0/47856/b05fdd22-6020-c6da-8e94-dc8f77e5513b.gif  
http://needtec.sakura.ne.jp/box2d_yunyaa/yukkuri.html  
  
  
 **LiquidFun**   
http://google.github.io/liquidfun/  
  
## 導入方法とサンプルコード  
以下からliquidfun-x.x.x.zipをダウンロードして任意のフォルダに解凍します。  
https://github.com/google/liquidfun/releases  
  
以下のフォルダがJavaScript関係のコードになります。  
liquidfun-1.1.0\liquidfun\Box2D\lfjs  
  
色々なコードが存在しますが、使うだけなら、「liquidfun.js」を以下のように取り込むだけで使えます。  
  
```html
<script type="text/javascript" src="liquidfun-1.1.0/liquidfun/Box2D/lfjs/liquidfun.js"></script>
```  
  
index.htmlならびにtestbed以下はサンプルコードになっています。  
LiquidFunはあくまで物理エンジンのライブラリであり、描画はおこなっていません。また、C++と異なり、JavaScript版にはデバッグ用の表示機能も有効になっていません。サンプルコードでは、three.jsを用いて描画を行っています。  
  
もし、three.jsを使う学習コストが高いようでしたら、下記のようにD3.jsで描画するといいでしょう。  
  
 **LiquidFunでJavaScript流体シミュレーション.**   
http://qiita.com/Quramy/items/578efec667267acf6871  
  
### サンプルコードの注意  
もし、LiquidFunに付属するthree.jsでなく、最新のthree.jsを用いてサンプルを動作させる場合には注意があります。  
renderer.js中のaddAttributeの呼び出し方法が古いので以下のように修正する必要があります。  
  
```js:liquidfun-1.1.0/liquidfun/Box2D/lfjs/testbed/renderer.js
function Renderer() {
  // init large buffer geometry
  this.maxVertices = 31000;
  var geometry = new THREE.BufferGeometry();
  geometry.dynamic = true;
  //addAttributeの形式が古い
  //geometry.addAttribute('position', Float32Array, this.maxVertices, 3);
  //geometry.addAttribute('color', Float32Array, this.maxVertices, 3);
  geometry.addAttribute('position', new THREE.BufferAttribute(new Float32Array(this.maxVertices * 3 ), 3));
  geometry.addAttribute('color', new THREE.BufferAttribute(new Float32Array(this.maxVertices * 3), 3));
```  
  
## JavaScriptの拡張  
使っているとわかるのですが、JavaScript版のLiquidFunはC++版の一部の機能しかサポートされていません。  
  
たとえば、粒子と剛体の接触イベントはC++ではサポートされています。  
http://google.github.io/liquidfun/API-Ref/html/classb2_contact_listener.html  
  
しかしながら、これらの機能は、JavaScriptではサポートされていません。  
JavaScriptでは剛体と剛体の接触しかイベントがコールされないのです。  
  
  
この場合、以下の手順でC++のコードをJavaScriptから利用できるようにする必要があります。  
  
https://qiita-image-store.s3.amazonaws.com/0/47856/b3d9760f-5ca6-24fb-b891-c68f080df3fd.png  
  
まず、emscriptenを用いてC++のコードをJavaScriptのコードに変換します。  
次に、複数のJavaScriptのファイルをClosureCompilerを用いて、１つのファイルに圧縮します。  
  
このビルドについての公式資料は下記を参考にしてください。  
https://google.github.io/liquidfun/Building/html/md__building_java_script.html  
  
ここでは、実際にDebian7上でliquidfunのJavaScriptをビルドしてみます。  
  
### C++のliquidfunをビルドする  
まず、C++版のliquidfunのコードをビルドします。  
C++でビルドするためには下記のライブラリーがインストール済みである必要があります。  
  
```
apt-get install cmake
apt-get install libglapi-mesa
apt-get install libglu1-mesa-dev
apt-get install libxi-dev
```  
  
その後は次のようにコマンドを実行してください。  
  
```
cd liquidfun/Box2D
cmake -G'Unix Makefiles'
make
make install
```  
  
### emscriptenの前準備  
emscriptenを動作させるのにnode.jsとPythonが必要なのでインストールします。  
PythonはLinux系ならデフォルトでインストールされているので、それを利用するといいでしょう。  
  
node.jsは下記の方法でインストールできます。  
  
 **Debian/UbuntuでNode.jsをインストールする(nvm)**   
http://qiita.com/tamurashingo@github/items/6348863668e1e3fd70c9  
  
  
### emscriptenの構築  
emscriptenはCやC++のコードをJavaScriptに変換します。  
たとえば、以下で紹介されているEM-DOSBOXはEmScriptenでMS-DOSをJavascript上で動作させています。  
  
 **Internet Archiveが2000以上のMS-DOSゲームをWebに移植、ブラウザで遊べる**   
http://jp.techcrunch.com/2015/01/10/20150109internet-archive-brings-oregon-trail-prince-of-persia-lemmings-and-2200-other-ms-dos-games-to-your-browser/  
  
emscriptenは一応クロスプラットフォーム対応なので、Windowsでも動作しますが、 **emsdk-1.29.0のインストーラーでインストールしたら環境変数をぶち壊してシステム復元をするはめになった** ので、おすすめしません。  
Windowsしかもってない方も、VMPlayer+Debian7で仮想環境を作れるのでそっちをお勧めします。  
  
そもそも、liquidfunの公式で「We use Emscripten on Linux, but you should be able to use the Emscripten SDK on Mac or Windows too, if you prefer. Note that Mac and Windows build environments have not been tested.」って言っているので、おとなしくLinux系でやったほうがいいでしょう。  
  
まず、下記のページから「Portable Emscripten SDK for Linux and OS X (emsdk-portable.tar.gz)」をダウンロードします。  
http://kripken.github.io/emscripten-site/docs/getting_started/downloads.html  
  
圧縮ファイルを取得後は次のコマンドを実行すればEmscriptenのビルドが行われます。  
  
```
gzip -dc emsdk-portable.tar.gz | tar xvf -
cd emsdk_portable
./emsdk update
./emsdk install latest
```  
  
この際に、clang/fastcompをダウンロードしてビルドをしていますが、この処理にはかなり時間がかかります。また、このビルドを中断して、再実行した場合は注意が必要です。  
emsdk installでは、フォルダの有無でインストール済みかを判断しているので、中途半端に処理を行っているとインストール済みとみなされます。  
その場合は手動でfastcomp/build_masterをmakeしなおしましょう。  
  
ビルドが完了したら、以下のコマンドを実行して使用するemsdkのバージョンと環境変数の登録を行います。  
  
```
./emsdk activate latest
source ./emsdk_env.sh
```  
  
### ClosureCompilerの構築  
ClosureComilerはJavascriptの圧縮を行うJavaで作成されたコマンドラインツールです。  
  
まず、以下から compiler.jarをダウンロードしてください。  
https://developers.google.com/closure/compiler/docs/gettingstarted_app  
  
その後、環境変数でCLOSURE_JARにcomiler.jarのパスを指定します。  
  
```
export CLOSURE_JAR=/share/closure/compiler.jar
```  
  
#### Javaのバージョン確認  
Javaのバージョンが7未満の場合、comiler.jarは以下のようなエラーを出力します。  
  
```
Exception in thread "main" java.lang.UnsupportedClassVersionError: com/google/javascript/jscomp/CommandLineRunner : Unsupported major.minor version 51.0
```  
  
この場合、Javaのバージョンを確認してみてください。  
  
```
java --version 
```  
  
DebianのJavaを7にあげるには以下の手順で行うと良いでしょう。  
  
 **Installing Java 7 (Oracle) in Debian via apt-get [closed]**   
http://stackoverflow.com/questions/15543603/installing-java-7-oracle-in-debian-via-apt-get  
  
```
echo "deb http://ppa.launchpad.net/webupd8team/java/ubuntu precise main" | tee -a /etc/apt/sources.list
echo "deb-src http://ppa.launchpad.net/webupd8team/java/ubuntu precise main" | tee -a /etc/apt/sources.list
apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys EEA14886
apt-get update
apt-get install oracle-java7-installer
```  
  
### liquidfunのJavaScriptをビルドする  
liquidfunのJavaScriptをビルドします。  
公式では下記のコマンドを実行するだけで作成できるといっていますし、実際作成できます。  
  
```
cd Box2D/lfjs
make
./uglify.sh
```  
  
しかし、ここで作成された、liquidfun.jsを使用するとエラーになります。  
この場合は、Makefileを修正します。  
  
```
$(EMSCRIPTEN)/emcc -I../ -o lf_core.js jsBindings/jsBindings.cpp $(OBJECTS) -s $(EXPORTS) -s TOTAL_MEMORY=33554432 -s ASSERTIONS=1 -O1 --js-library callbacks.js
```  
  
ここでは、-O2から-O1に変更しました。また、-s ASSERTIONS=1とすることで、エラーの詳細を出力できます。-O2の場合、以下のエラーが発生していました。  
  
```
"Assertion failed: you need to wait for the runtime to be ready (e.g. wait for main() to be called)"
```  
  
emscriptenは開発中のようで、安定しておらず、同様の自称が下記で報告されています。  
https://github.com/kripken/emscripten/issues/3082  
  
とりあえず、ここまでで、liquidfun.jsのビルドが行えるようになりました。  
既存のプログラムを新しいliquidfun.jsで動作させて動くことを確認しましょう。  
  
### 拡張のコツ  
ビルドまでも一苦労でしたが、実際、拡張するのも結構しんどいです。  
  
lfjs/jsBindings以下のコードをよく読んで、既存のオブジェクトがどのようにJavaScriptで実装されているかつかみましょう。  
  
  
#### offsets.jsを使用したクラス・構造体へのプロパティのアクセス方法  
jsBindings/offsets.js には各オブジェクトの構造体やクラスのプロパティについてのオフセット値が格納されています。ここで指定したオフセットはプロパティのアクセスに利用します。  
  
たとえば、b2ParticleContact構造体を実装する場合を例に挙げましょう。  
b2ParticleContactの構造体はC++では次のようになります。  
  
```cpp
struct b2ParticleContact
{
// 略
private:
    b2ParticleIndex indexA, indexB;

    float32 weight;

    b2Vec2 normal;

    uint32 flags;
};
// 略
```  
  
この構造体のweightを取得するJavaScriptコードは次のようになります。  
  
```js:lfjs/jsBindings/Particle/b2ParticleSystem.js
/**@constructor*/
function b2ParticleContact(ptr) {
  this.ptr = ptr;
  this.buffer = new DataView(Module.HEAPU8.buffer, ptr);
}

b2ParticleContact.prototype.GetWeight = function() {
  return this.buffer.getFloat32(Offsets.b2ParticleContact.weight, true);
};

```  
  
このようにoffset.jsで指定されたoffset値とオブジェクト作成時に与えられたポインターを利用して構造体のメンバーにアクセスしています。  
  
では、offset.jsのオフセット値はどのように指定するのでしょうか？  
実は、このオフセット値を指定するためのヒントが、lfjs/jsBindings/jsBindings.cppにあります。  
  
まず、lfjs/jsBindings/jsBindings.cppには次のようにコメントアウトされた行があるのでコレを有効にします。  
  
```cpp:lfjs/jsBindings/jsBindings.cpp
// TODO clean all of this up, and/or make it auto generated from the
// header files
void PrintOffsets(b2Body* b) {
  printf("\tb2Body: {\n");
  printf("\t\ttype: %u,\n", (unsigned int)&b->m_type - (unsigned int)b);
  printf("\t\tislandIndex: %u,\n", (unsigned int)&b->m_islandIndex - (unsigned int)b);
  printf("\t\txf: %u,\n", (unsigned int)&b->m_xf - (unsigned int)b);
  printf("\t\txf0: %u,\n", (unsigned int)&b->m_xf0 - (unsigned int)b);
  printf("\t\tsweep: %u,\n", (unsigned int)&b->m_sweep - (unsigned int)b);
  printf("\t\tlinearVelocity: %u,\n", (unsigned int)&b->m_linearVelocity - (unsigned int)b);
  printf("\t\tangularVelocity: %u,\n", (unsigned int)&b->m_angularVelocity - (unsigned int)b);
  printf("\t\tforce: %u,\n", (unsigned int)&b->m_force - (unsigned int)b);
  printf("\t\ttorque: %u,\n", (unsigned int)&b->m_torque - (unsigned int)b);
  printf("\t\tworld: %u,\n", (unsigned int)&b->m_world - (unsigned int)b);
  printf("\t\tprev: %u,\n", (unsigned int)&b->m_prev - (unsigned int)b);
  printf("\t\tnext: %u,\n", (unsigned int)&b->m_next - (unsigned int)b);
  printf("\t\tfixtureList: %u,\n", (unsigned int)&b->m_fixtureList - (unsigned int)b);
  printf("\t\tfixtureCount: %u,\n", (unsigned int)&b->m_fixtureCount - (unsigned int)b);
  printf("\t\tjointList: %u,\n", (unsigned int)&b->m_jointList - (unsigned int)b);
  printf("\t\tcontactList: %u,\n", (unsigned int)&b->m_contactList - (unsigned int)b);
  printf("\t\tmass: %u,\n", (unsigned int)&b->m_mass - (unsigned int)b);
  printf("\t\tinvMass: %u,\n", (unsigned int)&b->m_invMass - (unsigned int)b);
  printf("\t\tI: %u,\n", (unsigned int)&b->m_I - (unsigned int)b);
  printf("\t\tinvI: %u,\n", (unsigned int)&b->m_invI - (unsigned int)b);
  printf("\t\tlinearDamping: %u,\n", (unsigned int)&b->m_linearDamping - (unsigned int)b);
  printf("\t\tangularDamping: %u,\n", (unsigned int)&b->m_angularDamping - (unsigned int)b);
  printf("\t\tgravityScale: %u,\n", (unsigned int)&b->m_gravityScale - (unsigned int)b);
  printf("\t\tsleepTime: %u,\n", (unsigned int)&b->m_sleepTime - (unsigned int)b);
  printf("\t\tuserData: %u\n", (unsigned int)&b->m_userData - (unsigned int)b);
  printf("\t}\n");
}

void PrintOffsets(b2Contact* c) {
  printf("\tb2Contact: {\n");
  printf("\t\tflags: %u,\n", (unsigned int)&c->m_flags - (unsigned int)c);
  printf("\t\tprev: %u,\n", (unsigned int)&c->m_prev - (unsigned int)c);
  printf("\t\tnext: %u,\n", (unsigned int)&c->m_next - (unsigned int)c);
  printf("\t\tnodeA: %u,\n", (unsigned int)&c->m_nodeA - (unsigned int)c);
  printf("\t\tnodeB: %u,\n", (unsigned int)&c->m_nodeB - (unsigned int)c);
  printf("\t\tfixtureA: %u,\n", (unsigned int)&c->m_fixtureA - (unsigned int)c);
  printf("\t\tfixtureB: %u,\n", (unsigned int)&c->m_fixtureB - (unsigned int)c);
  printf("\t\tindexA: %u,\n", (unsigned int)&c->m_indexA - (unsigned int)c);
  printf("\t\tindexB: %u,\n", (unsigned int)&c->m_indexB - (unsigned int)c);
  printf("\t\tmanifold: %u,\n", (unsigned int)&c->m_manifold - (unsigned int)c);
  printf("\t\ttoiCount: %u,\n", (unsigned int)&c->m_toiCount - (unsigned int)c);
  printf("\t\ttoi: %u,\n", (unsigned int)&c->m_toi - (unsigned int)c);
  printf("\t\tfriction: %u,\n", (unsigned int)&c->m_friction - (unsigned int)c);
  printf("\t\trestitution: %u,\n", (unsigned int)&c->m_restitution - (unsigned int)c);
  printf("\t\ttangentSpeed: %u\n", (unsigned int)&c->m_tangentSpeed - (unsigned int)c);
  printf("\t},\n");
}

void PrintOffsets(b2Fixture* f) {
  printf("\tb2Fixture: {\n");
  printf("\t\tdensity: %u,\n", (unsigned int)&f->m_density - (unsigned int)f);
  printf("\t\tnext: %u,\n", (unsigned int)&f->m_next - (unsigned int)f);
  printf("\t\tbody: %u,\n", (unsigned int)&f->m_body - (unsigned int)f);
  printf("\t\tshape: %u,\n", (unsigned int)&f->m_shape - (unsigned int)f);
  printf("\t\tfriction: %u,\n", (unsigned int)&f->m_friction - (unsigned int)f);
  printf("\t\trestitution: %u,\n", (unsigned int)&f->m_restitution - (unsigned int)f);
  printf("\t\tproxies: %u,\n", (unsigned int)&f->m_proxies - (unsigned int)f);
  printf("\t\tproxyCount: %u,\n", (unsigned int)&f->m_proxyCount - (unsigned int)f);
  printf("\t\tfilter: %u,\n", (unsigned int)&f->m_filter - (unsigned int)f);
  printf("\t\tisSensor: %u,\n", (unsigned int)&f->m_isSensor - (unsigned int)f);
  printf("\t\tuserData: %u\n", (unsigned int)&f->m_userData - (unsigned int)f);
  printf("\t},\n");
}

void PrintOffsets(b2ParticleGroup* p) {
  printf("\tb2ParticleGroup: {\n");
  printf("\t\tsystem: %u,\n", (unsigned int)&p->m_system - (unsigned int)p);
  printf("\t\tfirstIndex: %u,\n", (unsigned int)&p->m_firstIndex - (unsigned int)p);
  printf("\t\tlastIndex: %u,\n", (unsigned int)&p->m_lastIndex - (unsigned int)p);
  printf("\t\tgroupFlags: %u,\n", (unsigned int)&p->m_groupFlags - (unsigned int)p);
  printf("\t\tstrength: %u,\n", (unsigned int)&p->m_strength - (unsigned int)p);
  printf("\t\tprev: %u,\n", (unsigned int)&p->m_prev - (unsigned int)p);
  printf("\t\tnext: %u,\n", (unsigned int)&p->m_next - (unsigned int)p);
  printf("\t\ttimestamp: %u,\n", (unsigned int)&p->m_timestamp - (unsigned int)p);
  printf("\t\tmass: %u,\n", (unsigned int)&p->m_mass - (unsigned int)p);
  printf("\t\tinertia: %u,\n", (unsigned int)&p->m_inertia - (unsigned int)p);
  printf("\t\tcenter: %u,\n", (unsigned int)&p->m_center - (unsigned int)p);
  printf("\t\tlinearVelocity: %u,\n", (unsigned int)&p->m_linearVelocity - (unsigned int)p);
  printf("\t\tangularVelocity: %u,\n", (unsigned int)&p->m_angularVelocity - (unsigned int)p);
  printf("\t\ttransform: %u,\n", (unsigned int)&p->m_transform - (unsigned int)p);
  printf("\t\tuserData: %u\n", (unsigned int)&p->m_userData - (unsigned int)p);
  printf("\t},\n");
}

void PrintOffsets(b2World* w) {
  printf("\tb2World: {\n");
  printf("\t\tbodyList: %u\n", (unsigned int)&w->m_bodyList - (unsigned int)w);
  printf("\t},\n");
}

void PrintOffsets(b2WorldManifold* wm) {
  printf("\tb2WorldManifold: {\n");
  printf("\t\tnormal: %u,\n", (unsigned int)&wm->normal - (unsigned int)wm);
  printf("\t\tpoints: %u,\n", (unsigned int)&wm->points - (unsigned int)wm);
  printf("\t\tseparations: %u\n", (unsigned int)&wm->separations - (unsigned int)wm);
  printf("\t},\n");
}

void PrintOffsets(b2ParticleContact* pc) {
  printf("\tb2ParticleContact: {\n");
  printf("\t\tindexA: %u,\n", (unsigned int)&pc->indexA - (unsigned int)pc);
  printf("\t\tindexB: %u,\n", (unsigned int)&pc->indexB - (unsigned int)pc);
  printf("\t\tweight: %u,\n", (unsigned int)&pc->weight - (unsigned int)pc);
  printf("\t\tnormal: %u,\n", (unsigned int)&pc->normal - (unsigned int)pc);
  printf("\t\tflags: %u\n", (unsigned int)&pc->flags - (unsigned int)pc);
  printf("\t},\n");
}

void PrintOffsets(b2ParticleBodyContact* pbc) {
  printf("\tb2ParticleBodyContact: {\n");
  printf("\t\tindex: %u,\n", (unsigned int)&pbc->index - (unsigned int)pbc);
  printf("\t\tbody: %u,\n", (unsigned int)&pbc->body - (unsigned int)pbc);
  printf("\t\tfixture: %u,\n", (unsigned int)&pbc->fixture - (unsigned int)pbc);
  printf("\t\tweight: %u,\n", (unsigned int)&pbc->weight - (unsigned int)pbc);
  printf("\t\tnormal: %u,\n", (unsigned int)&pbc->normal - (unsigned int)pbc);
  printf("\t\tmass: %u\n", (unsigned int)&pbc->mass - (unsigned int)pbc);
  printf("\t},\n");
}

# include <stdio.h>
extern "C" {
void GenerateOffsets() {
  printf("{\n");
  //create dummy body to generate offsets
  b2World world(b2Vec2(0, 0));
  b2BodyDef def;
  b2Body* body1 = world.CreateBody(&def);

  b2CircleShape s;
  b2FixtureDef d;
  d.shape = &s;
  b2Fixture* f =body1->CreateFixture(&d);

  b2ParticleSystemDef psd;
  b2ParticleSystem* ps = world.CreateParticleSystem(&psd);

  b2ParticleGroupDef pgd;
  b2ParticleGroup* pg = ps->CreateParticleGroup(pgd);

  b2WorldManifold worldManifold;
  PrintOffsets(body1);
  PrintOffsets(f);
  PrintOffsets(pg);
  PrintOffsets(&world);
  PrintOffsets(&worldManifold);

  b2ParticleContact pc;
  PrintOffsets(&pc);

  b2ParticleBodyContact pbc;
  PrintOffsets(&pbc);

  // need to instantiate contact differently
  //b2Contact contact;
  //PrintOffsets(&contact);

  printf("};\n");
}
```  
  
これはGenerateOffsetsを経由して、オフセット値を出力するための関数です。  
liquidfun.jsビルド後、Javascript側で下記のコードを実行すると、offset.jsに記述すべきオフセット値が出力されます。  
  
```js
console.log(_GenerateOffsets());
```  
  
 **・・・ってことをしたかたんでしょうが、実はこのコードはビルドすらできません。**   
それは当然で、liquidfunのC++のコードのプライベートのメンバーにアクセスしているからです。この場合、ビルド時にエラーが発生した箇所をかたっぱしからpublicに指定して、offset.jsを作成後元に戻す必要があります。  
  
  
#### 粒子の衝突イベントのサポート  
particleとparticleの衝突、particleとBodyの衝突イベントをサポートしたバージョンを下記に置きます。  
http://needtec.sakura.ne.jp/box2d_yunyaa/lfjs.zip  
  
Winmergeなりで、なにを替えたかを確認すれば、さらなる拡張のヒントになるかとおもいます。  
  
  
このイベントを利用するには、particleのflagsにb2_particleContactListenerParticle とb2_fixtureContactListenerParticleを付与します。  
  
```js
    var psd = new b2ParticleSystemDef();
    console.log(psd);
    psd.radius = particleRadius;
    this.particleSystem = world.CreateParticleSystem(psd);
    circle = new b2PolygonShape();
    circle.SetAsBoxXYCenterAngle(r, r, new b2Vec2(x, y), 0);
    pgd = new b2ParticleGroupDef();
    pgd.flags = b2_elasticParticle  | b2_particleContactListenerParticle | b2_fixtureContactListenerParticle ;
    pgd.groupFlags = b2_solidParticleGroup;
    pgd.shape = circle;
    this.particleSystem.CreateParticleGroup(pgd);
```  
  
その後、以下のようにコールバック関数を登録することで、該当フラグを付与したオブジェクトの衝突が検知できます。  
  
```js
    world = new b2World(gravity);
    var listener =  {
        BeginContactBody: function(contact) {
          console.log(contact.GetFixtureA());
        },
        EndContactBody: function(contact) {
            console.log(contact.GetFixtureA());
        },
        PostSolve: function(contact, impulse) {
            console.log('PostSolve');
        },
        PreSolve: function(contact, oldManifold) {

        },
        /**
         * ParticleオブジェクトとBodyの接触
         */
        BeginContactParticleBody : function(particleSystem, particleBodyContact) {
          console.log('----------------BeginContactParticleBody', 
            particleSystem,
            particleBodyContact.GetIndex(),
            particleBodyContact.GetNormal(),
            particleBodyContact.GetWeight(),
            particleBodyContact.GetMass(),
            particleBodyContact.GetBody(),
            particleBodyContact.GetFixture());
        },
        /**
         * Particleオブジェクト同士の接触
         */
        BeginContactParticle : function(particleSystem, particleContact) {
          console.log('----------------BeginContactParticleContact', particleSystem, particleContact.GetFlags(), particleContact.GetIndexA(), particleContact.GetIndexB(), particleContact.GetNormal(), particleContact.GetWeight());
        },
        EndContactParticleBody : function(fixture, particleSystem, index) {
          console.log('----------------EndContactParticleBody', fixture, particleSystem, index);
        },
        EndContactParticle : function(particleSystem, indexA, indexB) {
          console.log('----------------EndContactParticle', indexA, indexB);
        }
    }
    world.SetContactListener(listener);

```  
  
以下のサンプルのConsoleLogを監視してみてください。  
「ゆっくり」同士をぶつけた場合と離れた場合にメッセージが表示することが確認できます。  
  
http://needtec.sakura.ne.jp/box2d_yunyaa/yukkuri.html  
  
  
## まとめ  
LiquidFunを利用すると流体の物理演算をおこなってくれるので、流体やSoftBodyの表現がおこなえます。  
  
基本的にC++で100%の実力を発揮できますが、頑張ればJavaScriptでもなんとかなります。  
・・・とはいえ、くっそメンドクサイのです。  
