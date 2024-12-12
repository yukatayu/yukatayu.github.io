---
title: "空白の縁に立つ"
date: 2024-11-27T17:44:54+09:00
draft: false
categories: [ "日記" ]
tags: [ "言語", "表現", "願い" ]
image: "images/2024/VRChat_2022-08-29_11-55-22.924_3840x2160_resize.png"
type: post
summary: "計算機の上に在ってほしい地盤について考えます。"
---

# はじめに
あなたが見ている計算機の上には，本来幾つかの数値が存在するのみです。
そこに後から意味づけした結果を，あなたは見ています。

計算機の上に存在する意味というのは，大勢の手によって局所的に発展してきた存在です。
でも，そろそろ全く新たな地図を書いてみてもいい頃だと思うのです。

私が作りたいものは，見通しの効く世界です。

---

私が今までにこの手の話題に触れたのは，主に以下の三点です。

-  [同期に特化した言語を考えてみる（１）](/blog/sync_language_01)  
　——「システムの全体を見通せる状態で設計したい」という願い
-  [図書館を裏返す](/blog/vr_library_01)  
　—— 「情報の位置づけを明確にし，見通せるようにしたい」という願い
- このツイート  
<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">ロガーがI/Oを雑に発生させすぎ問題についてなのですが、私が欲しいのは、以下のような感じのOS or コンテナシステムなのです。<br>・プログラムは発生させうるエフェクト（≒システムコール）を型として持つ。(種類は雑に増やせる)<br>・OSの機能により、どのエフェクトをどのシステムコール or…</p>&mdash; ゆかたゆ (@yukata_yu) <a href="https://twitter.com/yukata_yu/status/1728419453183066183?ref_src=twsrc%5Etfw">November 25, 2023</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>  
　　　—— 「手元で流れている情報を見通し，制御したい」という願い

いずれも，見通しと可視化が願いの主軸です。

---

# 箱詰め

これらに関連した存在として，コンテナ技術があります。

私が普段コンテナに対して問題を感じているのは，以下のような点です。
- 外部から来たリクエストがコンテナ内に到達する過程が見えない（デバッグが困難）
- 各コンテナが発生させたり受け取ったりしうるメッセージの種類が分からず，型も付いていない
- コンテナ内にカスタマイゼーションポイントを置きづらい
- 再コンパイル時に，同一の機能を持つコンテナが生成される可能性が低い

これらは結局，既存のシステムを再利用するために詰め込んでいるからこそ発生している問題に見えます。

環境に依存するプログラムというものは，不透明な存在です。
その不透明な存在と環境を一つの箱に入れ，さらに巨大な不透明な存在にしてしまっているのが現代のコンテナ技術です。

ただの文字列よりも具体的な情報をやり取りし，そこに外界との契約が生じれば，システムの全体は見通せるようになります。
個々の不透明な存在による影響力を抑え込めば，システム全体は晴れ上がります。

ですが，このような考え方は，既に数十年前から言われていたはずの事です。
ただシステムを大きく作り直したくなかったがために，このような結果になってしまいました。
人類の開発力は有限なので仕方のない事とは言え，少し寂しい事ですね。

---

# 方針の整理

これらを前提として，「今欲しい物は Algebraic Effects ベースの OS ですか？」と問われると，悩んでしまいます。
少し欲張ってみて，ここまでに挙げた4つ（2記事＋1ツイート＋コンテナ）に共通の芯が通っていないかと考えてみましょう。

まずは，書き出してみます。

- 情報の流れの型付け
  - プログラムやコンテナが発生させる
  - 既存バイナリについてはアダプタを挟むか，FFIとしてアクセスする
- 接続の可視化と編集
  - OSやコンテナ管理層で実装する
  - 各プログラムやコンテナが発生させるエフェクトを他プログラムや外部システムへ再マッピング可能にする
- イベントにおいて契約による互換性の表明をすることで，中間層の入れ替えを可能にする
- 状態と影響の可視化
  - どのイベントが原因でどこの状態に影響を及ぼしたのかを可視化する
- 状態同士の繋がりとイベント同士のつながりを，両方とも可視化したい
  - 状態の間に存在する契約についても可視化したい
  - 物理的な接続と知識ネットワークを一つの空間上に表現したい

これらは基本的には「各プログラムの外側でも副作用を状態の一種として扱いたい」という願いです。
しかし，最後の「状態とイベントをグラフ上で並列に扱いたい」という願いは，開いたシステムに対してどの程度現実的か，今の時点では私にも想像が付いていません。

この部分が解決するまでは，過去記事で書いた「裏返された図書館」と「同期に特化した言語」と「進化したコンテナ」のうち，図書館だけは切り離された存在となります。

---

# まとめ

私が作りたい物は，以下の二つである気がしてきました。
- 発生するエフェクトとそれに関する契約を型として表明し，検証・可視化・繋ぎ変えを可能とする基盤
- 状態の繋がりと状態同士のつながり，それぞれを複数のレイヤーまたは観点で可視化できる表現方法

この二つは，きっといつか「初めから一つの存在の具体例であったもの」になると信じてはいます。
けれども，私がより良い抽象化を得るまでの間は別々に考える必要がありそうですね。

それまでは，それぞれの散歩道から見える景色を楽しむことにします。