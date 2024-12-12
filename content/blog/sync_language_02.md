---
title: "同期に特化した言語を考えてみる（２）"
date: 2024-09-03T10:39:00+09:00
draft: false
categories: [ "技術" ]
tags: [ "言語", "同期" ]
image: "images/2024/VRChat_2023-08-14_13-46-45.608_3840x2160_resize.png"
type: post
summary: "ネットワークシステムを拡張する際の事を考えてみます。"
---

## はじめに

この記事は，考え途中のアイデアをそのまま書き出したものです。  
自分でも考えをまとめ切れていないため，表現が分かりづらいかもしれません。

- 前回： [同期に特化した言語を考えてみる（１）](/blog/sync_language_01)
- 次回： まだです。
<!-- - 次回： [同期に特化した言語を考えてみる（３）](/blog/sync_language_03) -->

---

## 土地と建物と人

人は大抵，土地の上に立った家の中に住んでいます。その中に自分の好きな物を置いて，それらに囲まれながら生きています。

VRSNS等でも，プラットフォームが先に存在して，その上にワールドがあり，その中にアバターが居ると思います。

この順序をひっくり返すことは，全体の構成が変わらない限りは不可能です。

---

例えばVR空間で「部屋の中に\\(A\\)と\\(B\\)が居て，マグカップ \\(C_A, C_B\\) をそれぞれ持っている」という状況を考えてみます。

この時，部屋の子オブジェクトとして \\(\lbrace A, B, C_A, C_B \rbrace\\) がある場合と，子が\\(\lbrace A, B \rbrace\\) で孫が \\(\lbrace C_A, C_B \rbrace\\) の場合が考えられますね。

ワールド（⊃部屋）の子オブジェクトがアバターのみの場合，アイテムの挙動はアバターの一部として捉えられている筈です。
つまり，「アバター（↢プレイヤー）」が全てを包含するような存在として設計されていれば，それ以外のアイテムなどを取り扱うには特別な事は要らないはずです。

でも，それはいわゆる神クラスを作ることであり，ひいては何も定めないのと同じです。

対して，部屋の子オブジェクトとして \\(\lbrace A, B, C_A, C_B \rbrace\\) がある場合，それぞれの存在が個別に定義されますし，これは既存のVRSNS（e.g. VRChat）の構造にも近いので納得感があります。

---

## 重なる世界

では，新たな挙動をする存在をワールド上に存在させたいと思った時には，どのような処理が行われるべきでしょうか。

---

その前に，単一のネットワークについて考えてみましょう。

まず，値XとYがある時，これらの値は次の何れかで決定されます
- XとYは互いに無関係
- XはYの関数（Yが決まるとXが決まる）
- YはXの関数（Xが決まるとYが決まる）

---

「YとXが相互に依存している」という場面の考慮が必要になると思うかもしれませんが，それができる以上は，YからXへの変換が存在します。
従って，Xを自由変数にした上で常にXを変更することで，両方向の変換を後から実装できるということです。
言語機能として実装する必要は無いですので，この場では考慮しないことにします。

この時，「関数である」関係を有向グラフとした時，そのグラフは明らかに非巡回有向グラフとなります。
（巡回があると，互いに関係する2つの変数が存在してしまうため）

---

変数の関係が非巡回有向グラフとなる事実は，アイテムがどれだけ追加されても覆らない筈です。
ですが，一般のグラフを実行時に随時検査することはそれなりに難しく，何らかの追加制約が必要に思われます。

ここで先ほど挙げた「プラットフォーム→ワールド→アバターの順に存在する」という事実を踏まえると，グラフ同士の間に非巡回有向グラフを設定し，依存元グラフは依存先グラフの全ての変数を読み取れるようにします。

この時，変数は自明に非巡回有向グラフになります。

さらに，「プラットフォームだけ」「プラットフォームとワールドだけ」でもシステムが存在できるようになります。
変数寿命管理の観点でも有利に働きそうですね。

ただし，外部との通信を抽象化する部分は，必ずその森の根になります。書き込みエフェクト発生がデータ送信ですね。

---

これらの多重ネットワークによって，
- 複数のレイヤーにおける非巡回の依存グラフ
- 複数の横並びの存在における依存グラフ
- これらを接続するための，同じノードタイプのインスタンス間の通信
を実装していきます。

---

## 表現

これらを表現する場合でも，基本的には前回記事と同じ表現で出来ると思います。

素朴には，参加者の定義と，そのグラフを持っておきます。
グラフ同士には権限設定用の弱順序があり，最小のグラフをただひとつ持ちます。
プラットフォームは，その最小のグラフをエントリーポイントとして起動します。

それから，前回何の説明もなく `b.hp` などのプロパティ変数を出してしまいました。
このように，各変数は｢この参加者に対して1つ定義される｣というlocalityを持つはずです。
localityを参加者の定義とくっつけても本当に良いのかについても，そのうち考えたいですね。
もしかしたら，所有権の移転などもできると嬉しいかもしれませんので。

---

## 書き込む

ここまで依存グラフの存在について触れてきましたが，実際に影響を起こす時の事を考えてみます。

依存グラフの方向に影響を伝播させるときには，値を単に参照すれば良いです。

対して，依存グラフを遡るようなエフェクトを発生させるときには，時間の差を考える必要があります。
基本的には，エフェクトが発生した時点でベクタークロックを付けてリクエストを送り，何らかの検査後に作用を起こせば良さそうです。

流石に記事が長くなってしまうので，この話題は次回に回そうと思います。

---

あとは何についての考慮が足りていないのでしょうか？ ……atomic性とロールバックですね。

道のりは長そうですけれど，次回以降考えてみます。

それではまた。