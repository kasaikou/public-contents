---
title: 【技育CAMP参戦記録 - 開発高速化編】rehearsal【アイデア/振り返り編】
emoji: 🎉
type: idea
topics: [ハッカソン, 振り返り, アイデア, 個人開発]
published: false
---

<!--todo: イベント名を確認する-->
今回は2021年07月04日に行われた「技育CAMP vol.4 開発を高速化」に参加した際に見たアイデア視点での振り返りです。
技術視点での振り返りは[「個人開発で使用した技術の話 vol.2【rehearsal】](introduce_rehearsal_02)をご覧ください。


# 簡単に自己紹介とそもそもどんなものを作ったのかの話
## 3項目でまとめる自己紹介

- 19才の大学2年です。
- おそらく、ボクの自己紹介は文章でまとめるよりもTwitterを見に行ったが早い気がするのでそちらを見てください。今は大学で衛星打ち上げようかな～、ロボット作ろうかな～、と右往左往しながら毎日過ごしています。
- **Twitterはフォローしておくと過去の経歴だけでなく未来の経歴も見ることができるのでフォローしておいたほうが良いです。**

@[card](https://twitter.com/streamwest1629)

## 3項目でまとめる制作物紹介

- **rehearsal** という名前のコンソール上で実行可能なテストツールを作りました。
- 複数のアプリケーションの標準入出力同士のリアルタイムなパイプを提供します。
- 今回は深堀しませんが、`golang` を使用して作りました。技術選定の理由などは「個人開発で使用した技術の話 vol.2【rehearsal】（技育展2021が終わるまでお待ちください）をご覧ください。
<!-- - 今回は深堀しませんが、`golang` を使用して作りました。技術選定の理由などは[「個人開発で使用した技術の話 vol.2【rehearsal】](introduce_rehearsal_02)をご覧ください。 -->

# モチベーション
## 開発中
構想は一か月ほど前からあったのですが、実際に作り始めると何をどうすればいいのかわからず、ということが多々ありました。それだけではなく、実際に作り始める三日ほど前に[「技育CAMP vol.4 無駄開発をしよう](introduce_desires-of-sheep_01)のプレゼンを行った結果、成果物として申し分なかったにもかかわらず、プレゼンのために賞を逃してしまったこともあり、前回よりも多くの時間をプレゼン準備に割きたいと思いました。
とはいうものの、実際の開発は思うようにはなかなか進んでくれず、結局実装したかった機能をすべて実装せずに、プレゼンで最低限必要な部分の開発にとどまりました。

## 発表準備中
前回のハッカソンでプレゼンで失敗したことを踏まえて、前回よりも多くの時間をプレゼン準備に割きました。VOICELOIDの琴葉葵を用いた動画でプレゼン時間の尺を稼ぎつつ、わかりやすく、モダンなプレゼン資料を作ることだけを意識しました。もっと言えば、今回のプロダクトがコンソールアプリケーションということもあり、その古めかしい部分を動画の力で覆い隠すかが重要である、そういう認識で持って取り組みました。

## 発表中・ほかの発表を見て
発表順でボクは一番最初の登壇になりました。前回同様かなり緊張しながらとはいえ、つよつよな人たちに囲まれることなく、自分の発表ができたと思います。ただ、発表とはいっても半分以上はVOICELOIDによるデモ動画の再生だったので、すべて口頭による発表にしたとき、同じ発表ができるかどうかといわれると今のボクも自信がないです。
ほかのチームが制作した成果物について、ほかのハッカソンとは比べ物にならないくらい優秀すぎる作品が多すぎてビビりました。多くのチームでチーム開発を効率化するようなプロダクトを制作してきて、個人開発ばかりしてるボクではどうしても想像しにくいものがあったと思っています。
結果として、今回制作したrehearsalは努力賞を受賞することができました。わーい。

# 編集後記
前回、ハッカソン参戦記録として記事にした[「【技育CAMP参戦記録 - 無駄開発編】希望の睡眠【アイデア/振り返り編】」](introduce_desires-of-sheep_01)と同じくらいの文章量になるように書いたのですが、話の密度としてはこっちのほうが濃いような気がします。
色々と思い出しながら書いた記事ですが、記憶力がないのでそこにあった事実しか覚えておらず、何を思ってそういう行動に出たのか、という自分の思考の部分が抜けているように思いました。
制作物を作っている間もしっかり何を思ったのか書き留めておかないとダメそうです。

https://docs.google.com/presentation/d/1BZcwHe4nWJIgxGl1mR7c9_kgf23wZgf0JXtp0F24t0I/edit?usp=sharing