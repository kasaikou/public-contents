---
title: JupyterLabを動かそう後編 - docker-compose.ymlを書く
---

# そうだ，Docker Composeに頼ろう

この回では前回書いたdockerfileを実際にコンテナとして実行するために，docker-compose.ymlを書いていきます．
[公式ドキュメント](https://docs.docker.com/compose/)では，Docker Composeは以下のように説明されています．
> Compose is a tool for defining and running multi-container Docker applications.

ComposeはYAMLファイルを使って一個以上のコンテナの設定を行うことができ，`docker compose`コマンドの一回の実行で設定されたコンテナの管理を行うことができます．

今回はDocker Composeを使ってコンテナの実行を管理していくために，docker-compose.ymlを書きます．

# 全体像を見る
記述するファイルの量が少ないので，今回は最初に全体像を確認しましょう．
```yaml
version: "3.8"
services:
  jupyterlab:
    build:
      context: .docker
    command: python3 -m jupyter lab --ip=0.0.0.0 --port 8888 --allow-root
    ports:
      - 8888:8888
```

# ポイント
## version
```yaml
version: "3.8"
```
docker-composeでバージョン情報です．この段階ではあまり気にならないかもしれませんが後でGPUを対応させる際にDocker Compose v3.8（厳密にはDocker Engineリリースがv19.03）以降が求められるはず（記憶違いだったらごめんなさい）なので，3.8にしてあります．なぜかは知りませんが，文字列である必要があるので注意してください．

## services
```yaml
services:
  jupyterlab:
```
ここでコンテナを定義します．`jupyterlab`という名前のコンテナを一個定義します．

## build
```yaml
jupyterlab:
  build:
    context: .docker
```
ビルドはコンテナをビルドする際に用いる設定です．色々設定項目があるので公式ドキュメントを読んでもらった方が良いのですが，ここでは `context` でdockerfileの場所をdocker-compose.ymlからの相対パスで示しています．

## command
```yaml
command: python3 -m jupyter lab --ip=0.0.0.0 --port 8888 --allow-root
```
コンテナを開始する際に実行するコマンドです．Jupyterlabをポート指定で実行しています．これがないとDockerはコンテナを作っても何もしません．

## port
```yaml
ports:
  - 8888:8888
```
Dockerコンテナと紐づけるローカルのポートを指定します（ドキュメントだと「公開用ポート」と表現されるやつです）．

`(ホスト側のポート):(コンテナ側のポート)`という形で指定することができ，Jupyterlabをはじめとするウェブアプリケーション（Jupyterがウェブアプリケーションかという疑問はさておき）に対してホスト側のポートからアクセスすることができるようになります．
