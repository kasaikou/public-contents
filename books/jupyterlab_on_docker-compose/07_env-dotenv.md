---
title: Docker Composeとローカルの話前編 - .envと環境変数
---

# .envと環境変数
前回まで実際にDockerやDocker Composeを使ってコンテナを作っていました．この回からは趣向を変えて，ローカル環境とコンテナをつなげるための方法を見ていきたいと思います．

この回では各環境ごとに変えるべき環境変数と，コンテナ内で設定するべき共通の環境変数をどのように設定するかを確認します．

# .envの使い方
環境変数を定義する際に最初に連想するものは`.env`ではないでしょうか．Docker Composeを扱う際には二種類の.envファイルがあります．

1. docker-compose.yml内で参照するために定義する環境変数
2. コンテナ内で用いるために定義する環境変数

それぞれについて深掘りしていきます．

## docker-compose.yml内で参照するために定義する環境変数
Docker Composeでは，docker-compose.yml内で`${DOTENV_KEYNAME}`という形で変数を設定することができます．

どちらかと言えば，docker-compose.ymlの外部から値を読み取るという表現の方がよいかもしれません．Docker Compose内がローカル環境によって場合分けをしたい場合の手段と言えます．
既定ではdocker-compose.ymlと同じディレクトリ内にある.envファイルからKey-Valueペアを読み取ります．

## コンテナ内で用いるために定義する環境変数
Dockerコンテナ内で用いることができる環境変数をdocker-compose.yml内で設定することができます．

設定方法は二種類あり，docker-compose.ymlに直接記述する方法と，前者同様，.envファイルとして環境変数のリストを与える方法です．両方の方法で行うことができ，それぞれ複数の環境変数や複数のリストを与えることができます．両方に同じキーが与えられている場合は，docker-compose.ymlに直接記述されている方が優先されます．

以下は例です．
```yaml
jupyterlab:
  env_file:
    - .docker/.env
  environment:
    ENVIRONMENT: prod
    API_KEY: ${API_KEY}
```

:::message
最後の`API_KEY`環境変数は，前者のdocker-compose.yml内で参照するために定義された環境変数を参照する形で設定しています．

こうすることで複数のコンテナを建てる場合であっても，確実に共通の環境変数を割り当てることができます．
:::