---
title: JupyterLabを動かそう中編 - Dockerfileを書く
---

# そうだ，Dockerfileを書こう
この回では実際にdockerfileを書いていきます．
dockerfileはコンテナイメージを作るためのファイルで，`apt-get`等々を用いてLinux環境を構築していきます．

少し悩んだのですが，今回はJupyterLabと合わせて`jupyterlab-lsp`も一緒に追加します．

## FROM
ベースとなるコンテナイメージを指定します．本来は`python:3.9`など（すでにPythonが動作するコンテナイメージ）を指定するのですが，後からGPU対応コンテナにすることを考慮し，あえて`ubuntu:18.04`を指定します．

```dockerfile
FROM ubuntu:18.04
```

## `DEBIAN_FRONTEND`
[この解説](https://qiita.com/udzura/items/576c2c782adb241070bc)によれば，環境変数`DEBIAN_FRONTEND`に`nointeractive`を指定するとインストール時の設定でインタラクティブな設定（入力待ち）をしなくなります．
Dockerはコンテナビルド時にインタラクティブな設定はできません．入力待ちがあるとその場で来ることのない入力を待ち続けることになります．

環境変数をDockerfile内で指定するときは`ENV key=value`の形で指定します．
一緒に`TZ=Asia/Tokyo`を指定しておきます．

```dockerfile
ENV DEBIAN_FRONTEND=nointeractive
ENV TZ=Asia/Tokyo
```

## 必要なツールを`apt-get install`
コンパイラなどの必要なツールを`apt-get install`します．本当は今いらないものもかなりあるのですが，後から追加するのも面倒なので（と言いながらなんか追加することになりそう）極力使いそうなものは先に入れておきます．
例えば`zlib1g-dev`と`libjpeg-dev`は今構築している段階では使いませんが，Pythonの代表的なライブラリである`matplotlib`を使用する際の必須ツールとなります．

Dockerfileでコマンドを実行する際には`RUN command --args`という具合で実行していきます．バックスラッシュを使って複数行にすることができたり，`&&`で複数のコマンドの順次実行などができます．
イメージサイズを削減する観点から，複数のコマンドをまとめるのがDockerのベストプラクティスとされています．

最初の行
```dockerfile
# sed -i.bak -r 's!(deb|deb-src) \S+!\1 mirror://mirrors.ubuntu.com/mirrors.txt!' /etc/apt/sources.list && \
```

これはDockerfileのコメントアウトされた行ですが，このコメントアウトを外すと`apt-get install`する際にミラーを使うようになります．
インタラクティブな操作ができないので，`(y/n)?`と聞かれないように`-y`フラグを付けておきましょう．

```dockerfile
RUN \ 
    # sed -i.bak -r 's!(deb|deb-src) \S+!\1 mirror://mirrors.ubuntu.com/mirrors.txt!' /etc/apt/sources.list && \
    apt-get update && \
    apt-get install -y \
    tzdata \
    curl \
    dirmngr \
    apt-transport-https \
    lsb-release \
    ca-certificates \
    build-essential \
    git \
    wget \
    tar \
    python3 \
    python3-pip \
    zlib1g-dev \
    libjpeg-dev \
    graphviz
```

## nodejsをインストール
必要なさそうですが，`jupyterlab-lsp`をインストールする際に求められるので一緒にインストールします．
とはいっても，nodejsはaptに登録されていないので，追加します．方法の解説に関してはいろいろな方が **「Ubuntuにnodejsをインストールする方法」** として解説しているので割愛（さっきのミラーを含め，コピペで済ませている部分がかなりある（なんでBookに書こうと思ったのか））

```dockerfile
RUN curl -sL https://deb.nodesource.com/setup_14.x | bash - && \
    apt-get install -y nodejs
```

## JupyterLabをインストール
`jupyterlab`と`jupyterlab-lsp`をインストールします．以下の三点が主なポイントです．

1. 予め`dockerfile`と同じ階層に`requirements.txt`を追加しておきます．
    - 今回は`jupyterlab`と`jupyterlab-lsp`さえあれば十分です．
    - 最終的には`jupyterlab`と`jupyterlab-lsp`のほかにも「どんなプロジェクトでも使うようなライブラリ」を入れておくことを想定しています．
    - `numpy`や`matplotlib`のほかにも，私は`jupyterlab-git`や`jupyterlab_code_formatter`，`yapf`，`ipython-sql`などを入れています．

2. (1)で作った`requirements.txt`をビルド中のコンテナに入れます．
    - `WORKDIR`でカレントディレクトリを移動します．`cd`コマンドみたいなものです．
    - Dockerfileでは `ADD src_path dest_path`の形でローカルからコンテナ内へファイルやディレクトリをコピーすることができます．コピー元，コピー先それぞれ色々な指定方法があるので詳しい解説は[公式ドキュメント](https://docs.docker.com/engine/reference/builder/#add)や[日本語ドキュメント](https://matsuand.github.io/docs.docker.jp.onthefly/engine/reference/builder/#add)を読んでください．
3. `python3`で実行してください．

```dockerfile
WORKDIR /tmp/build/jupyterlab
ARG python_lsp_version=3.9.1
ADD requirements.txt .
RUN python3 -m pip install -r requirements.txt && \
    python3 -m jupyter labextension install --no-build \
    "@krassowski/jupyterlab-lsp" && \
    python3 -m jupyter lab build
```
