---
title: Stable Diffusionを使って絵の依頼を出してボカコレ2022秋に投稿した
emoji: 🖊️
type: idea
topics: [stablediffusion, AI, 画像生成, ボカコレ2022秋]
published: true
---

# ボカコレ2022秋にご参加の皆様，いかがお過ごしでしょうか
皆様，曲はちゃんと予定通り投稿できたでしょうか．私は曲を水曜日あたりから作り始めて土曜日の午前零時に投稿したので正味3日で曲と動画を作りました．その前日までボカコレで投稿する予定なんてなく，曲のアイデアを思いついたのですぐに形にしたら思いのほかテンポよくできてしまったので，今回急いで動画を作成し投稿するに至りました．

https://www.nicovideo.jp/watch/sm41193969

今回はその際にStable Diffusionを通して絵の依頼を行ったのでその一部始終の話を残しておこうと思い，執筆することにしました．

:::message
一応，対象読者は創作者視点にしているつもりなんですが，まあZennに書いている時点でエンジニアにもほんの少しだけ読んでほしいなという気持ちでかいていたりします．

創作者に比べるとエンジニアは人の心
:::

# 依頼するまでの一部始終
初めの章で書いたとおり，今回は曲を作り始めてから公開するまでの日数が3日と，とにかく時間がありませんでした．そこで，今回私は一つの可能性としてStable Diffusionを用いてイラストを作成することを検討しました．

**計画性がない！**

## Stable Diffusionとは
Stable Diffusion[^stablediffusion]は文章から画像を生成する画像生成モデル（俗にいうAI）です．同様のことを行う有名な画像生成モデルに，「DALLE2[^dalle2]」，「Imagen[^imagen]」，「Midjourney[^midjourney]」，「NovelAI Diffusion[^novelaidiffusion]」などがありますが，その中でもStable Diffusionはデータセット・モデルについての情報が無償公開されているモデルです．

[^stablediffusion]: https://github.com/CompVis/stable-diffusion
[^dalle2]: https://openai.com/dall-e-2/
[^imagen]: https://imagen.research.google/
[^midjourney]: https://www.midjourney.com/home/
[^novelaidiffusion]: https://novelai.net/

私の過去にStable Diffusionで制作した作品群を紹介します．

![](https://storage.googleapis.com/zenn-user-upload/7123246c12fd-20221009.jpg)

（ちなみに，私がStable Diffusionを使っているのはローカルで実行できる安心感が主な理由です）

## Stable Diffusionを動画のイラストとして使うことのリスク
考察のところで詳しく解説しますが，私にStable Diffusionを使った絵を動画に使うための勇気がありませんでした．幸い，私には2年半くらいの付き合いがあって信頼を置いている絵師のもこね（[@777_mocone](https://twitter.com/777_mocone)）さんがいるので，土下座をしてでも依頼する方が無難だという判断を下しました．

:::message
二年半の付き合いについては，[「今年の総括(2020-2021)とコミュニティ」]という記事で軽く~~触れている~~触れていませんがとても面白いのでよろしければ読んでみてください．（とても長い記事なので無理はなさらずに）
:::

[「今年の総括(2020-2021)とコミュニティ」]: https://zenn.dev/streamwest1629/articles/annual_summary-2020to2021

私はStable Diffusionを使用することで絵師さんからの反感を買うことを何より警戒していました．今回の曲がStable Diffusionを動画に使用する絵として採用する価値のある，例えば「人工物」みたいな曲のテーマであればいざ知らず，ただ「工数を削減する」という目的で使用することに私個人として抵抗がありました．

とはいえ，ただでさえ日数が少ないのは確かなので十分もこねさんに断られてしまう可能性はありました．そこで，私は絵の雰囲気をStable Diffusionで生成していました．

そうしてプロンプトを捻りながら生成したイラストがこちらになります．
![](https://storage.googleapis.com/zenn-user-upload/81ed968e5bb1-20221009.png)

しばらく経って，もこねさんから依頼の承諾をいただいたので，以下のような文言で依頼を出しました．

> 上のイラストをもこねさんっぽく書いてくれればOKです
> 割と服とかそこまでこだわりはないので書きやすいように書いてもらえると🥺

これが今回の，Stable Diffusionを使って絵の依頼を出す，という新しい試みの一部始終でした．

![](https://storage.googleapis.com/zenn-user-upload/dc7f18fe638d-20221011.jpg)

# 考察

ここからは，あくまで私と私が見聞した意見です．色々な意見があってしかるべきだと私は思うので，よろしければコメントやツイートで記事に対してレコメンドをいただけると幸いです．ただし，私がそれらに反応するとは限りませんのでその点ご了承ください．

:::message alert
また，私を含めて個人に対してDMなどで直接言及しに行く行為はご遠慮ください．
:::

## 私の第一印象

まずはじめに，絵をいただいたときに私が抱いた感想として，今回暗い曲に合う絵の依頼だったことを踏まえても，普段見るもこねさんの絵柄ではなかったように感じた，ということが挙げられます．

https://twitter.com/777_mocone/status/1559853420114513920

https://twitter.com/777_mocone/status/1480524791803432962

これらはもこねさんが普段Twitterで公開しているイラストですが，この通り，元々明るい絵を描く方だったので実際に依頼で書いていただいた絵を見たときにかなり驚きがありました．

結論としては，今回はそれがいい方向に働いたように思います．しかし，裏を返せば，依頼したその絵師さんの個性が参考資料として渡すStable Diffusionの生成画像に引っ張られる可能性があるので一概に見栄えのある絵が参考資料として良いとは限らないかもしれません．場合によっては，雑な構図を手書きで書いてそれを渡した方が絵師さんの個性が反映される余地があってよい場合があることを示唆しているように感じました．

## もこねさんの感想

ここで，もこねさんから頂いた，この絵の依頼に対する感想を見ていきます．

> 人にもよるかもしれないですが、真っ白な所から描くのが苦手なタイプなので、もとある絵を自分らしく直すやり方は、とっても描きやすかったです。
> 今回は時間がなかったので逆に有難かったのですが、普段だったら入れたいモチーフやイメージだったり、伝えたいこと、女の子のしぐさや表情についてもう少し情報があればもっと描き込めたかもしれないですね。

今回，私はStable Diffusionで生成した絵を渡して，「上のイラストをもこねさんっぽく書いてくれればOKです」というかなり雑な絵の依頼の仕方をしました．
それにもかかわらず，目指すべき絵の方針を提供するという目的はStable Diffusionを用いることでかなり効率的に行えていたことがもこねさんの感想からうかがえます．
一方で，生成画像を渡したからと言って「この絵の中の何をモチーフとして扱うべきか」「この絵からどのような変更を加えるべきか」を伝えなくてもいいかといわれるとそうではなく，生成画像を渡したそのうえでどういう方針の絵なのかなど，意思疎通を図る方がお互いのイメージがしっかり共有されるように感じました．

## 「モチーフはあったほうがいい」とは限らない

ここまではモチーフとしてStable Diffusionを用いる話をしました．

ところで私は，趣味で動画制作の依頼を受けたりするのですが，事前にリクエストがあると「このリクエストに対応した物を作らなければいけないのか」というつらみになりがちなのですが，どうやら絵師さんの中にも同じタイプがいるという話を頂きました．

つまり，ある程度リクエストに応えられる（言い方を変えれば依頼として受けている）絵師さんと，一から自分の思った通りの絵を書きたい絵師さんがいるということです．そのため，前者の方々には好まれる一方，後者の方々には不快感を与えてしまう可能性があることを頭の片隅に置いておいたほうがよさそうです．これは生成画像を使わなくても同じことが言えますが，前述したとおり生成画像は口頭で伝える情報よりもより細かい情報を与えてしまうので一層気を付けた方がよいでしょう．

## 本番使用としてのStable Diffusion

今回の一件の中で起こりうるシナリオの中に「Stable Diffusionで生成したイラストをそのまま動画として使用する」という可能性がありましたが，それは私が何より避けたい未来でした．

https://twitter.com/StreamWest1629/status/1577947696211849217

私の本当に個人的な感想ですが，実際に画像生成を本番用に使用することでそれによって起こり得るリスクが高いうちは，よっぽど「この曲のテーマは画像生成で生成した絵のほうが絶対によい」という確信でもない限りは普通に依頼するんだろうな，と思いました．

# まとめ

今回の一件はStable Diffusionが公開されてから検討していたシナリオの一つだったので，実際に運用できたことがまず第一歩という感じです．

https://twitter.com/StreamWest1629/status/1561601885399748608

かなり思った通り明確なイメージの共有が行えたことはよかったと思います．ただ，モチーフとして使用するにも少し絵がきれいなだけで銀の弾丸ではないのでちゃんとコミュニケーションはしなければいけなかったなど，反省点も多かったように感じました．

最後になりますが，私が強く信頼しているもこねさんへ一言．
**翌日締め切りの依頼出して本当にごめんなさい**

そしてこれを読んだすべての創作者へ．
**翌日締め切りの依頼に限らず制作期間詰め詰めスケジューリングなんて絶対にやめましょう**

以上になります．最後までありがとうございました．