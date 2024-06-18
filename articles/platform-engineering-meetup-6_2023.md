---
title: Platform Engineering の最小単位について
emoji: 🤔
type: tech
topics: [Platform Engineering]
published: true
---

今回の技術記事は「Platform Engineering Meetup #6」で話を聞いてきて知らない人なりに感じたことをまとめてみる回です．たぶん狭義の Platform Engineering とは微妙にずれているのですが，まあそこは見逃して頂ければなと思います．

結論，別に社内サービスでなくても GitOps や ChatOps のような既存の考え方であっても Platform Engineering なのでは？と思ったのがここで言いたいことです．

そもそも， Platform Engineering について説明すると「クラウドネイティブ時代において、ソフトウェアエンジニアリング組織にセルフサービス機能を提供するためのツールチェーンやワークフローを設計・構築する技術分野（[Connpass の Platform Enginnering Meetup #6](https://platformengineering.connpass.com/event/299834/)引用）」ということです．背景的には，ソフトウェア技術の進化によってコンテナやら監視やらセキュリティやらとそれぞれに特化した様々な概念を良い感じにまとめ上げるセルフサービスのことだと認識しています．

これは，直近でも感じる所があり，例えば今ワタシの所属している研究室は最先端技術を取り扱っていると確信していますが，情報系ではないので Git すら使えないわけです．こういう所にコンテナやらなにやらを導入すればすぐに爆発してしまいます．だからそれをうまく抽象化して，単純な操作だけで扱えるようにしなければいけない，というニーズが出てきます．（これが悪いと言うわけではないですが）実験装置の操作一つとっても最初にしっかり PowerPoint でひとりひとりがマニュアルを書いて装置の破壊や事故を防いでいく，という状況においては Git は職人芸のような難しさがあるのです．

実際，リクルートの方の発表では 機械学習エンジニアが機械学習に集中できるような社内サービスを構築したり，ヤフーの方の発表でもサービス開発者向けの社内サービスを構築しているという話がありました．

ただ，当たり前ですがそんなことを構築する人的リソースの余裕のない組織だって五万とある中，そんな社内サービスによる Platform Engineering というのは絵に描いた餅なわけです．

ここで，結論なのですが，別に社内サービスでなくても GitOps や ChatOps のような既存の考え方であっても Platform Engineering なのでは？という所に戻ってきます．
前述したように，Git を使えない組織もたくさんあるのであくまで情報系サービス業に限った話になるのですが，だいたいのソフトウェアエンジニア（組み込みエンジニアもですが）の共通で使用できるツールとして Git は十分な立ち位置にあり，これを軸にして開発した方が安上がりなうえに社内で使用できるツールとして障壁が低いのです．

さて， GitOps や ChatOps が Platform Engineering となるために，前述の「セルフサービス機能を提供するためにツールチェーンやワークフローを設計・構築していく」ということなのですが，単に CI を組むだけではこれを満たすには不足であると感じます．なぜなら，この「設計・構築」が1サービスに対してのアプローチではなく継続的にアプローチする必要があるからです．言い換えれば，一つのワークフローを何個ものリポジトリが共有している，という状況でなければ Platform Engineering として社内サービスと比較したときどうしても弱く見えるのです．

ということでここまでの話を基に最後に一つ仮説を出して終わろうと思います．他力本願ですが，合っているかどうかの検証記事は誰か書いてください．

「ソフトウェアエンジニアにおける Platform Engineering の最小構成はリポジトリ間で共有される GitHub Actions のアクションないし Circle CI の orbs などで構成された共通 CI である（これが一番小さいと思います）」

短いですが以上となります．

おしまい