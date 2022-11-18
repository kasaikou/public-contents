---
title: VSCode Devcontainer 放浪記
emoji: ☂
type: tech
topics: [VSCode, Devcontainer]
published: false
---

[かさい]: https://twitter.com/streamwest1629
[VSCode Advent Calendar 2022]: https://qiita.com/advent-calendar/2022/vscode

# Docker でローカルテストを行っている皆様，いかがお過ごしでしょうか

皆様おはようございます．[かさい]です．今年一年， VSCode の Devcontainer 様々な所でコケまくったり，色々な知見があったので Devcontainer について知識整理を行い，ついでにその経験を [VSCode Advent Calendar 2022] に奉納することにしました．

よろしければ Advent Calendar の他の方々の記事も見ていってください．

この記事では，Docker　についての事前情報がある方を対象に， Devcontainer とはどういうものなのか，なぜそれを使いたがるのか，一般的な Docker コンテナの構築指針とどういった違いがあるのかを整理していきます．

Devcontainer をすでに使っている方にも助けとなるような記事を目指して書きましたので最後まで見ていってもらえればと思います．

# TL;DR
- Devcontainer は**メリットがいっぱい**だよ
    - ローカル環境が汚れないし**再現性のある開発環境ができる**よ
    - **リポジトリ単位で使用する環境を変える**ことができるよ
    - 開発者全員に同じ開発環境を強制できるよ
- Devcontainer は**デメリットも無視できない**よ
    - 通常のコンテナ以上に**メモリを消費**するよ
- Devcontainer を気持ちよく運用するにはコツがいるよ
    - 開発環境だとしても，ビルドやプルで時間がかかるのは論外だからね
    - 対してキャッシュは積極的に使えるよ
    - **root で動かすなシバくぞ**

# Devcontainer とは
[^1]: https://code.visualstudio.com/docs/devcontainers/containers

Devcontainer (本来は `Dev Containers` と表記するみたいですね[^1]）は VSCode の機能で開発環境として作られたコンテナの中で開発することができます．近年，Docker や Kubernetes, 各種 IaaS のおかげでサーバーの実行環境をコンテナとすることが一気に増えましたが，コンテナを開発環境に用いることができます．と言っても，実行環境と開発環境が異なるように，実行用の Dockerfile と開発用の Dockerfile ももちろん異なります．

[Dev Containers tutorial]: https://code.visualstudio.com/docs/devcontainers/tutorial

基本的には以下のようなファイル構成です．`Dockerfile` と `docker-compose.yml` は `devcontainer.json` の設定に応じて記述します． VSCode のテンプレート（[Dev Containers tutorial] で示されている方法で作成することができます）で用意されるは `devcontainer.json` と `Dockerfile` で構成されていて，その2ファイルがあればだいたい使えるのですが，正直 `docker-compose.yml` 書いた方が見通し良いので基本的に私は書いています．

:::message
`Dockerfile` と `docker-compose.yml` に関してはこのディレクトリでなくても問題はありません（どのみち `devcontainer.json` でパスを指定するため）が，実行用の Dockerfile と混同するのを防ぐ観点からまとめておくに越したことないです．
:::

```
. (repo directory)
├ .devcointainer
│ ├ devcontainer.json (required)
│ ├ Dockerfile (optional)
│ ├ docker-compose.yml (optional)
│ └ ...
└ ... (source files)
```

イメージとしては，開発環境としてビルドされたコンテナにリポジトリのあるディレクトリをボリューム（バインドマウント）としてマウントして起動し， VSCode でマウントしたディレクトリをコンテナ経由で開く，というイメージです．ですので， GUI としては 通常の VSCode と変わりません．

もちろんターミナルはビルドしたコンテナに依存します．

![](https://storage.googleapis.com/zenn-user-upload/249cf9d6d64d-20221118.png)
*devcontainer についての図 [^1]*

## メリット1: ローカル環境が汚れない

一つ目のメリットは，ローカル環境が汚れません．コンテナ内で

## メリット2: リポジトリ単位で使用する環境を変えることができる

## メリット3: 開発環境に再現性が生まれる

## デメリット1: メモリをいっぱい食べます

# Devcontainer を用意する意義（大事）

# 通常のコンテナと異なる点

## アンチパターン1: root で実行する（するな）


## アンチパターン2: ライブラリのダウンロードを行う（行うな）

## 積極的にキャッシュを使う

# 基本構成（2022-11-18 現在におけるワタシの場合）

私は基本， Docker Compose を用いて構築します．理由は簡単で， devcontainer.json だけで実用に足りるまで持っていくよりも docker-compose.yml ベースに記述して必要なところだけ devcontainer.json で記述する方が楽，というだけです．

ただ，毎度 1 から作ると面倒な時も多いのでその時は適当に VSCode のテンプレートで組み合わせて使います．

~~私は Golang で開発することが多いので Golang で開発することを前提にしてみます．~~

個人開発で作成したリポジトリの中で一番出来のよさそうな Terraform 運用用のリポジトリ（プライベートリポジトリです）から引っ張ってきたので Terraform がメインです．なぜか Golang も入っているので許してください．

## devcontainer.json
```json5
// For format details, see https://aka.ms/devcontainer.json. For config options, see the README at:
// https://github.com/microsoft/vscode-dev-containers/tree/v0.241.1/containers/docker-from-docker-compose
{
    "name": "Docker from Docker Compose",
    "dockerComposeFile": "docker-compose.yml",
    "service": "devcontainer",
    "workspaceFolder": "/workspace",
    // Use this environment variable if you need to bind mount your local source code into a new container.
    "remoteEnv": {
        "LOCAL_WORKSPACE_FOLDER": "${localWorkspaceFolder}"
    },
    // Configure tool-specific properties.
    "customizations": {
        // Configure properties specific to VS Code.
        "vscode": {
            // Add the IDs of extensions you want installed when the container is created.
            "extensions": [
                "mhutchie.git-graph",
                "eamodio.gitlens",
                "EditorConfig.EditorConfig",
                "hashicorp.terraform",
                "ms-azuretools.vscode-docker"
            ]
        }
    },
    // Use 'forwardPorts' to make a list of ports inside the container available locally.
    // "forwardPorts": [],
    // Use 'postCreateCommand' to run commands after the container is created.
    // "postCreateCommand": "docker --version",
    "postAttachCommand": ".devcontainer/postAttach.sh",
    // Comment out to connect as root instead. More info: https://aka.ms/vscode-remote/containers/non-root.
    "remoteUser": "vscode"
}
```

## Dockerfile
```Dockerfile
FROM ubuntu:20.04

LABEL maintainer="Kasai Kou"

ARG apt_get_server=ftp.jaist.ac.jp/pub/Linux
ARG golang=1.18
ARG terraform=1.2.6
ARG username=vscode
ARG useruid=1000
ARG usergid=${useruid}

ENV DEBIAN_FRONTEND=nointeractive \
    LANG=ja_JP.UTF-8

WORKDIR /opt
RUN \
    #
    # apt-get
    sed -i s@archive.ubuntu.com@${apt_get_server}@g /etc/apt/sources.list && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
    curl \
    apt-transport-https \
    libarchive-tools \
    ca-certificates \
    git \
    zip \
    unzip \
    wget \
    sudo \
    && \
    #
    # create user
    groupadd --gid ${usergid} ${username} && \
    useradd -s /bin/bash --uid ${useruid} --gid ${usergid} -m ${username} && \
    echo ${username} ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/${username} && \
    chmod 0440 /etc/sudoers.d/${username} && \
    #
    # install golang
    wget -qO- "https://go.dev/dl/go${golang}.linux-$(dpkg --print-architecture).tar.gz" | tar zxf - -C /usr/local && \
    #
    # install AWS CLI
    curl -qo- "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" | bsdtar xvf - && \
    chmod +x ./aws/install && \
    ./aws/install && \
    chmod +x /usr/local/bin/aws && \
    #
    # install AWS Session Manager plugin
    curl "https://s3.amazonaws.com/session-manager-downloads/plugin/latest/ubuntu_64bit/session-manager-plugin.deb" -o "session-manager-plugin.deb" && \
    dpkg -i session-manager-plugin.deb && \
    #
    # install Terraform
    wget -qO- https://releases.hashicorp.com/terraform/${terraform}/terraform_${terraform}_linux_$(dpkg --print-architecture).zip | bsdtar xvf - -C /usr/local/bin/ && \
    chmod +x /usr/local/bin/terraform && \
    curl -s https://raw.githubusercontent.com/terraform-linters/tflint/master/install_linux.sh | bash

ENV GOPATH=/home/${username}/go
ENV PATH=${PATH}:/usr/local/go/bin
ENV PATH=${PATH}:/go/bin
USER ${username}
```

## docker-compose.yml
```yml
version: "3.8"

services:
  devcontainer:
    build:
      context: ..
      dockerfile: .devcontainer/Dockerfile
    volumes:
      - ..:/workspace:cached
      - gohome:/home/vscode/go:cached
    command: /bin/sh -c "while sleep 1000; do :; done"
    user: vscode
volumes:
  gohome:
```

## postAttach.sh
```sh
#!/bin/sh

cd `dirname $0`
cd ..
sudo chown -R vscode ~

# install go development kit
go install golang.org/x/tools/gopls@latest
go install golang.org/x/lint/golint@latest
go install github.com/go-delve/delve/cmd/dlv@master
go install github.com/haya14busa/goplay/cmd/goplay@v1.0.0
go install github.com/fatih/gomodifytags@v1.16.0
go install github.com/josharian/impl@latest
go install github.com/cweill/gotests/gotests@latest
go install github.com/ramya-rao-a/go-outline@latest
go install golang.org/x/tools/cmd/godoc@latest
go install honnef.co/go/tools/cmd/staticcheck@latest

go mod download
```