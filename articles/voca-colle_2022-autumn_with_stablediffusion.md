---
title: Stable Diffusionを使って絵の依頼を出した話
emoji: 🖊️
type: idea
topics: [stablediffusion, AI, 画像生成]
published: false
---

# ボカコレ2022秋にご参加の皆様，いかがお過ごしでしょうか
皆様，曲はちゃんと予定通り投稿できたでしょうか．私は曲を水曜日あたりから作り始めて土曜日の午前零時に投稿したので正味3日で曲と動画を作りました．その前日までボカコレで投稿する予定なんてなく，曲のアイデアを思いついたのですぐに形にしたら思いのほかテンポよくできてしまったので，今回急いで動画を作成し投稿するに至りました．

https://www.nicovideo.jp/watch/sm41193969

今回はその際にStable Diffusionを通して絵の依頼を行ったのでその一部始終の話を残しておこうと思い，執筆することにしました．

:::message
一応，対象読者は創作者視点にしているつもりなんですが，まあZennに書いている時点でエンジニアにも読んでほしいなという気持ちでかいていたりします．
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

私の過去の作品群を紹介します．

![](https://storage.googleapis.com/zenn-user-upload/7123246c12fd-20221009.jpg)

（ちなみに，私がStable Diffusionを使っているのはローカルで実行できる安心感が主な理由です）

## Stable Diffusionを動画のイラストとして使うことのリスク
結論から言ってしまうと，私にStable Diffusionを使った絵を動画に使うための勇気がありませんでした．幸い，私には2年半くらいの付き合いがあって信頼を置いている絵師のもこね（[@777_mocone](https://twitter.com/777_mocone)）さんがいるので，土下座をしてでも依頼する方が無難だという判断を下しました．

:::message
二年半の付き合いについては，[「今年の総括(2020-2021)とコミュニティ」]という記事で軽く触れているのでよろしければ読んでみてください．（とても長い記事なので無理はなさらずに）
:::

[「今年の総括(2020-2021)とコミュニティ」]: https://zenn.dev/streamwest1629/articles/annual_summary-2020to2021

私はStable Diffusionを使用することで絵師さんからの反感を買うことを何より警戒していました．今回の曲がStable Diffusionを動画に使用する絵として採用する価値のある，例えば「人工物」みたいな曲のテーマであればいざ知らず，ただ「工数を削減する」という目的のために使うことに私個人として抵抗がありました．

とはいえ，ただでさえ日数が少ないのは確かなので十分もこねさんに断られてしまう可能性はありました．そこで，私は絵の雰囲気をStable Diffusionで生成していました．

そうしてプロンプトを捻りながら生成したイラストがこちらになります．
![](https://storage.googleapis.com/zenn-user-upload/81ed968e5bb1-20221009.png)

しばらく経って，もこねさんから依頼の承諾をいただいたので，以下のような文言で依頼を出しました．

> 上のイラストをもこねさんっぽく書いてくれればOKです
> 割と服とかそこまでこだわりはないので書きやすいように書いてもらえると🥺

これが今回の，Stable Diffusionを使って絵の依頼を出す，という新しい試みの一部始終でした．

# 考察

