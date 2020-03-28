せっかくのクリスマスなのでマタイによる福音書のイエズス・キリストの系図でも描いてみます。  
  
```
アブラハムの子はイサク、
イサクの子はヤコブ、
ヤコブの子はユダと、その兄弟たち、
ユダの子はタマルによるペレズと、ゼラ、
ペレズの子はヘズロン、
ヘズロンの子はラム、
ラムの子はアミナダブ、
アミナダブの子はナション、
ナションの子はサルモン、
サルモンの子はラハブによるボアズ、
ボアズの子はルツによるオベデ、
オベデの子はエッサイ、
エッサイの子はダビデである。

ダビデの子はウリヤの妻によるソロモン、
ソロモンの子はレハバアム、
レハバアムの子はアビヤ、
アビヤの子はアサ、
アサの子はヨシャパテ、
ヨシャパテの子はヨラム、
ヨラムの子はウジヤ、
ウジヤの子はヨタム、
ヨタムの子はアハズ、
アハズの子はヒゼキヤ、
ヒゼキヤの子はマナセ、
マナセの子はアモン、
アモンの子はヨシヤ、
ヨシヤの子はバビロン追放当時の、エコニヤと、その兄弟たちである。

バビロン追放の後は次の通り。
エコニヤの子はシャルテル、
シャルテルの子はゼルバベル、
ゼルバベルの子はアビウデ、
アビウデの子はエリヤキム、
エリヤキムの子はアゾル、
アゾルの子はサドク、
サドクの子はアキム、
アキムの子はエリウデ、
エリウデの子はエレアザル、
エレアザルの子はマタン、
マタンの子はヤコブ、
ヤコブの子はマリアの夫のヨセフである。
```  
  
## 準備  
Cabocha 0.68のインストール  
http://qiita.com/mima_ita/items/161cd869648edb30627b  
  
GraphVizとpydotのインストール  
http://needtec.exblog.jp/20608517/  
  
## ユーザ辞書をつくる  
固有名があるので、形態素解析ができるようにユーザ辞書を作成します。  
以下のようなCSVをつくります。  
  
```bible.csv
アブラハム,0,0,500,名詞,一般,*,*,*,*,アブラハム,アブラハム,アブラハム
イサク,0,0,500,名詞,一般,*,*,*,*,イサク,イサク,イサク
ヤコブ,0,0,500,名詞,一般,*,*,*,*,ヤコブ,ヤコブ,ヤコブ
ユダ,0,0,500,名詞,一般,*,*,*,*,ユダ,ユダ,ユダ
ゼラ,0,0,500,名詞,一般,*,*,*,*,ゼラ,ゼラ,ゼラ
ペレズ,0,0,500,名詞,一般,*,*,*,*,ペレズ,ペレズ,ペレズ
ヘズロン,0,0,500,名詞,一般,*,*,*,*,ヘズロン,ヘズロン,ヘズロン
ラム,0,0,500,名詞,一般,*,*,*,*,ラム,ラム,ラム
アミナダブ,0,0,500,名詞,一般,*,*,*,*,アミナダブ,アミナダブ,アミナダブ
ナション,0,0,500,名詞,一般,*,*,*,*,ナション,ナション,ナション
サルモン,0,0,500,名詞,一般,*,*,*,*,サルモン,サルモン,サルモン
ボアズ,0,0,500,名詞,一般,*,*,*,*,ボアズ,ボアズ,ボアズ
オベデ,0,0,500,名詞,一般,*,*,*,*,オベデ,オベデ,オベデ
エッサイ,0,0,500,名詞,一般,*,*,*,*,エッサイ,エッサイ,エッサイ
ダビデ,0,0,500,名詞,一般,*,*,*,*,ダビデ,ダビデ,ダビデ
ソロモン,0,0,500,名詞,一般,*,*,*,*,ソロモン,ソロモン,ソロモン
レハバアム,0,0,500,名詞,一般,*,*,*,*,レハバアム,レハバアム,レハバアム
アビヤ,0,0,500,名詞,一般,*,*,*,*,アビヤ,アビヤ,アビヤ
アサ,0,0,500,名詞,一般,*,*,*,*,アサ,アサ,アサ
ヨシャパテ,0,0,500,名詞,一般,*,*,*,*,ヨシャパテ,ヨシャパテ,ヨシャパテ
ヨラム,0,0,500,名詞,一般,*,*,*,*,ヨラム,ヨラム,ヨラム
ウジヤ,0,0,500,名詞,一般,*,*,*,*,ウジヤ,ウジヤ,ウジヤ
ヨタム,0,0,500,名詞,一般,*,*,*,*,ヨタム,ヨタム,ヨタム
アハズ,0,0,500,名詞,一般,*,*,*,*,アハズ,アハズ,アハズ
ヒゼキヤ,0,0,500,名詞,一般,*,*,*,*,ヒゼキヤ,ヒゼキヤ,ヒゼキヤ
マナセ,0,0,500,名詞,一般,*,*,*,*,マナセ,マナセ,マナセ
アモン,0,0,500,名詞,一般,*,*,*,*,アモン,アモン,アモン
ヨシヤ,0,0,500,名詞,一般,*,*,*,*,ヨシヤ,ヨシヤ,ヨシヤ
バビロン,0,0,500,名詞,一般,*,*,*,*,バビロン,バビロン,バビロン
エコニヤ,0,0,500,名詞,一般,*,*,*,*,エコニヤ,エコニヤ,エコニヤ
シャルテル,0,0,500,名詞,一般,*,*,*,*,シャルテル,シャルテル,シャルテル
ゼルバベル,0,0,500,名詞,一般,*,*,*,*,ゼルバベル,ゼルバベル,ゼルバベル
アビウデ,0,0,500,名詞,一般,*,*,*,*,アビウデ,アビウデ,アビウデ
エリヤキム,0,0,500,名詞,一般,*,*,*,*,エリヤキム,エリヤキム,エリヤキム
アゾル,0,0,500,名詞,一般,*,*,*,*,アゾル,アゾル,アゾル
サドク,0,0,500,名詞,一般,*,*,*,*,サドク,サドク,サドク
アキム,0,0,500,名詞,一般,*,*,*,*,アキム,アキム,アキム
エリウデ,0,0,500,名詞,一般,*,*,*,*,エリウデ,エリウデ,エリウデ
エレアザル,0,0,500,名詞,一般,*,*,*,*,エレアザル,エレアザル,エレアザル
マタン,0,0,500,名詞,一般,*,*,*,*,マタン,マタン,マタン
ヤコブ,0,0,500,名詞,一般,*,*,*,*,ヤコブ,ヤコブ,ヤコブ
```  
  
Windowsの場合は次のコマンドをコマンドプロンプトで実行するとbible.dicというユーザ辞書が作成できます。  
  
```
"C:\Program Files (x86)\MeCab\bin\mecab-dict-index" -d"C:\Program Files (x86)\MeCab\dic\ipadic" -u bible.dic -f shift-jis -t utf-8 bible.csv
```  
  
Cabochaでは-uオプションでユーザ辞書のパスを指定することで、ユーザ辞書を使用した係り受け解析が行えます。  
  
## 家系図作成用のコード  
  
```py:bible.py
# !/usr/bin/python
# -*- coding: utf-8 -*-
import CaboCha
import pydot
import uuid
import sys


class node:
    """
    家系図のノードを記録
    """
    def __init__(self):
        self.name = None
        self.children_node = {}
        self.pairs_node = {}
        self.id = str(uuid.uuid4())

font_name = "ms ui gothic"

sentence = """
アブラハムの子はイサク、\n
イサクの子はヤコブ、\n
ヤコブの子はユダと、その兄弟たち、\n
ユダの子はタマルによるペレズと、ゼラ、\n
ペレズの子はヘズロン、\n
ヘズロンの子はラム、\n
ラムの子はアミナダブ、\n
アミナダブの子はナション、\n
ナションの子はサルモン、\n
サルモンの子はラハブによるボアズ、\n
ボアズの子はルツによるオベデ、\n
オベデの子はエッサイ、\n
エッサイの子はダビデである。\n

ダビデの子はウリヤの妻によるソロモン、\n
ソロモンの子はレハバアム、\n
レハバアムの子はアビヤ、\n
アビヤの子はアサ、\n
アサの子はヨシャパテ、\n
ヨシャパテの子はヨラム、\n
ヨラムの子はウジヤ、\n
ウジヤの子はヨタム、\n
ヨタムの子はアハズ、\n
アハズの子はヒゼキヤ、\n
ヒゼキヤの子はマナセ、\n
マナセの子はアモン、\n
アモンの子はヨシヤ、\n
ヨシヤの子はバビロン追放当時の、エコニヤと、その兄弟たちである。\n

バビロン追放の後は次の通り。\n
エコニヤの子はシャルテル、\n
シャルテルの子はゼルバベル、\n
ゼルバベルの子はアビウデ、\n
アビウデの子はエリヤキム、\n
エリヤキムの子はアゾル、\n
アゾルの子はサドク、\n
サドクの子はアキム、\n
アキムの子はエリウデ、\n
エリウデの子はエレアザル、\n
エレアザルの子はマタン、\n
マタンの子はヤコブ、\n
ヤコブの子はマリアの夫のヨセフである。\n
"""


def get_word(tree, ix):
    surface = tree.token(ix).surface
    f = tree.token(ix).feature.split(",")
    return surface, f


def create_node(tree, chunk):
    s1, f1 = get_word(tree, chunk.token_pos + chunk.head_pos)
    s2 = None
    f2 = None
    type = 0
    if chunk.head_pos != chunk.func_pos:
        s2, f2 = get_word(tree, chunk.token_pos + chunk.func_pos)
        if f2[0] == '助詞':
            if f2[1] == '連体化':  # 「の」
                type = 1
            elif f2[1] == '係助詞':  # 「は」
                type = 2
            elif f2[1] == '接続助詞':  # 「と」
                type = 3
            elif f2[1] == '格助詞':  # 「による」
                type = 4
    if f1[0] == '名詞':
        if f1[1] == '接尾':
            s1 = tree.token(chunk.token_pos + chunk.head_pos - 1).surface + s1
    return {
        'text': s1,
        'type': type
    }


def find_child(t, child_name):
    """
    家系図から指定の子を持つノードを取得する
    """
    for key, item in t.children_node.items():
        if key == child_name and item is None:
            return t
        elif item is not None:
            ret = find_child(item, child_name)
            if ret:
                return ret
    return None


def dump_family(family_tree):
    """
    家系図のダンプ
    """
    print ('親：', family_tree.name)
    for key, item in family_tree.pairs_node.items():
        print ('妻：', key)

    for key, item in family_tree.children_node.items():
        print ('子：', key)
        if item:
            print (dump_family(item))


def create_graph_children(graph, p_node, child):
    n = pydot.Node(child.id,
                   label=child.name,
                   style="filled",
                   fillcolor="green",
                   shape = "box",
                   fontname=font_name)
    subg = pydot.Subgraph('', rank='same')
    subg.add_node(n)
    graph.add_subgraph(subg)
    if p_node:
        graph.add_edge(pydot.Edge(p_node, n))

    for key, item in child.pairs_node.items():
        pair = pydot.Node(str(uuid.uuid4()),
                          label=key,
                          style="filled",
                          fillcolor="pink",
                          shape = "box",
                          fontname=font_name)
        subg.add_node(pair)
        graph.add_edge(pydot.Edge(n, pair))

    for key, item in child.children_node.items():
        if item:
            create_graph_children(graph, n, item)
        else:
            nchild = pydot.Node(str(uuid.uuid4()),
                                label=key,
                                style="filled",
                                fillcolor="gray",
                                shape = "box",
                                fontname=font_name)
            graph.add_node(nchild)
            graph.add_edge(pydot.Edge(n, nchild))


def create_graph(family_tree):
    graph = pydot.Dot(graph_type='digraph',
                      fontname=font_name)
    create_graph_children(graph, None, family_tree)
    graph.write_png('example2_graph.png')


def main(argvs, argc):
    c = CaboCha.Parser("-u bible.dic")
    lines = sentence.split("\n")
    family_tree = None
    for line in lines:
        tree = c.parse(line)
        data = node()
        if tree.chunk_size() == 0:
            continue
        ids = {-1: None}
        children = []
        pairs = []
        for i in range(tree.chunk_size()):
            curid = i
            if i in ids:
                continue
            chunk = tree.chunk(i)
            attr = None
            cnn_type = 0
            while True:
                d = create_node(tree, chunk)
                ids[curid] = d
                if data.name is None:
                    data.name = d['text']
                if cnn_type == 1:  # 前のチャンクが「の」で終わっている
                    if d['text'] == '子':
                        attr = 'children'
                elif cnn_type == 2:  # 前のチャンクが「は」で終わっている
                    if attr == 'children':
                        children.append(curid)
                elif cnn_type == 3:  # 前のチャンクが「と」で終わっている場合は、同じ操作
                    if attr == 'children':
                        children.append(curid)
                else:
                    attr = None
                if d['type'] == 4:  # チャンクが「よる」で終わっている場合は、配偶者とみなす
                    pairs.append(i)
                else:
                    cnn_type = d['type']
                if not chunk.link in ids:
                    curid = chunk.link
                    chunk = tree.chunk(curid)
                else:
                    break
        for child in children:
            data.children_node[ids[child]['text']] = None

        for pair in pairs:
            data.pairs_node[ids[pair]['text']] = None

        if len(children) > 0:
            if family_tree is None:
                family_tree = data
            else:
                node_obj = find_child(family_tree, data.name)
                if node_obj:
                    node_obj.children_node[data.name] = data
    if family_tree:
        dump_family(family_tree)
        create_graph(family_tree)


if __name__ == '__main__':
    argvs = sys.argv
    argc = len(argvs)
    sys.exit(main(argvs, argc))
```  
  
これを実行すると次のような家系図が生成されます。  
![example2_graph.png](/image/11b14ac1-755c-2acd-079b-c55389b0676a.png)  
  
同じような文体で記述すれば、多分、いろんな家系図が作成できまます。  
  
一応、配偶者を複数設定できるデータ構造なのは、ご先祖様のただれた・・・もとい精力的な交配活動に対応できるようにしてますが、誰と誰の子ってのは意識してません。  
