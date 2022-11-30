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

皆様おはようございます．[かさい]です．今年一年， VSCode の Devcontainer の様々な所でコケまくったり，色々な知見があったので Devcontainer について知識整理を行い，ついでにその経験を [VSCode Advent Calendar 2022] に奉納することにしました．

よろしければ Advent Calendar の他の方々の記事も見ていってください．

この記事では，Docker　についての事前情報がある方を対象に， Devcontainer とはどういうものなのか，なぜそれを使いたがるのか，一般的な Docker コンテナの構築指針とどういった違いがあるのかを整理していきます．

Devcontainer をすでに使っている方にも助けとなるような記事を目指して書きましたので最後まで見ていってもらえればと思います．

# TL;DR
- Devcontainer は**メリットがいっぱい**だよ
    - ローカル環境が汚れないし**再現性のある開発環境ができる**よ
    - **リポジトリ単位で使用する環境を変える**ことができるよ
    - 開発者全員に同じ開発環境を提供できるよ
- Devcontainer は**デメリットも無視できない**よ
    - 通常のコンテナ以上に**メモリを消費**するよ
- Devcontainer を気持ちよく運用するにはコツがいるよ
    - 開発環境だとしても，ビルドやプルで時間がかかるのは論外だからね
    - 対してキャッシュは積極的に使えるよ
    - **root で動かすなシバくぞ**
    - できる人はアーキテクチャも気にしてあげよう

# Devcontainer とは
[^1]: https://code.visualstudio.com/docs/devcontainers/containers

Devcontainer (本来は `Dev Containers` と表記するみたいですね[^1]）は VSCode の機能で開発環境として作られたコンテナの中で開発することができます．近年，Docker や Kubernetes, 各種 IaaS のおかげでサーバーの実行環境をコンテナとすることが一気に増えましたが，そのコンテナを開発環境として用いることを目的とした機能です．と言っても，実行環境と開発環境が異なるように，実行用の Dockerfile と開発用の Dockerfile ももちろん異なります．

[Dev Containers tutorial]: https://code.visualstudio.com/docs/devcontainers/tutorial

基本的には以下のようなファイル構成です．`Dockerfile` と `docker-compose.yml` は `devcontainer.json` の設定に応じて記述します． VSCode のテンプレート（[Dev Containers tutorial] で示されている方法で作成することができます）は `devcontainer.json` と `Dockerfile` で構成されていて，その2ファイルがあればだいたい使えるのですが，正直 `docker-compose.yml` 書いた方が見通し良いので基本的に私は書いています．

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

基本的にメリットはコンテナを運用する場合と同じではあるのですが，開発環境特有のメリットもあるので次はそれを見ていきます．

## メリット1: ローカル環境が汚れない

一つ目のメリットは，ローカル環境が汚れません．コンテナ内で開発に必要なツールをすべてそろえるので，ローカルに置いておく必要がなくなります．このメリットによって，開発環境で競合などの不都合が発生しても簡単に引き戻しを行うことができ，ローカル環境には VSCode と Docker さえあれば，あとは開発環境に必要なツールを稼働することができるようになるのです．

## メリット2: リポジトリ単位で使用する環境を変えることができる

通常，リポジトリ内に Devcontainer 用の Dockerfile などで定義されたファイルに従って開発環境が構築されるので，リポジトリ単位で使用する環境を簡単に変えることができます．リポジトリで使用されている技術の性質（ウェブアプリケーションであるか，スマホアプリであるかなど）によって開発環境を調整することができるのです．

また，Devcontainer 用の Dockerfile を Git をはじめとしたバージョン管理システムの管理下に置くことで，開発環境をチームで簡単に共有・更新することができるのです．

## メリット3: 開発環境に再現性が生まれる

Dockerfile で定義されたコンテナは（ベースとする OS イメージのアップデートなどもあるので絶対的であるとは言いませんが）ある程度の再現性があります．これによって複数人・複数デバイスで開発したとしても同一のマシンで開発している状態に限りなく近い状態とすることができるのです．

上記のことに付随して，開発環境作成に用いた Dockerfile がそのまま環境構築のドキュメントとして機能します．私の個人的な問題ではあるのですが，私は Linux を使うのがあまり得意ではないので，開発用のツールをインストールするためにどのようなコマンドを使用すればよいのか，使用するツールのバージョン，ファイルの位置や環境変数の値などについても明文化させておくことができるというのは大きなアドバンテージとなるのです．

---

## デメリット1: 大量のメモリを消費する

VSCode Server が無視できない程度に大量のメモリを消費します．

![](https://storage.googleapis.com/zenn-user-upload/9b8c2e07f643-20221119.png)
*記事執筆中のコンテナの stats*

![](https://storage.googleapis.com/zenn-user-upload/6d2f7f15e119-20221119.png)
*記事執筆中のコンテナにおけるプロセス別のメモリ使用量*

# Devcontainer を用意する意義（大事）

これまで， Devcontainer を使用するメリットについて整理しましたが，結局のところ何のために用いるのかという点について整理していきます．

結局のところ，最大のメリットはプロジェクトに参加する開発者全員の開発環境を一定に揃えられることにあります．これは開発者が他にどのようなプロジェクトに参加しているか，あるいはどのような環境のマシンを使用しているかに関わらず，そのプロジェクト専用の開発環境を利用することができ，どのようにして構築された開発環境であるかを確認できるのです．

プロジェクト参加時の環境構築と構築した環境の更新を行う場面で強い力を発揮することは想像に難しくありません．プロジェクト参加時の環境構築は VSCode と Docker （Git はホストマシンにもあったほうが便利であるものの）さえあればよく，あとは VSCode 内で Devcontainer に接続するだけです．更新する場合にも，更新箇所を Dockerfile で修正してリポジトリなどで共有し，それをもとに再ビルドを行うだけなのです．これほど簡単なことはありません．

# 通常のコンテナと異なる点

さて，ここまで Devcontainer がいかに素晴らしい開発環境を提供するかについて整理しました．次に確認すべきは，通常のコンテナ，つまり実行環境としてのコンテナと比較して開発環境として整える Devcontainer の Dockerfile にはどのような差異があるかについてです．

## ホストマシン内のファイルをコンテナ内で書き換える

考えるまでもなく `1=1` の当たり前な話ですが，コンテナにボリュームとしてマウントして，コンテナ内のユーザーを用いて VSCode からファイルを編集していくので，ホストマシン内のファイルをコンテナ内で書き換えています．これはどのような問題を引き起こすかというと，保存を行ったコンテナにおいて， VSCode がコンテナの `root` ユーザーでファイルの保存を行った場合， `root` として保存され，上書きされたファイルのアクセス権限はホストマシンにも適用されます．

### アンチパターン: root で実行する（するな）

つまるところ，言いたいことはこれです． `root` でコンテナ内に入ってはいけません． `root` で実行した場合，ホストマシンでファイルを変更することが困難になります．これは `.git` ディレクトリ内のファイルを含めたリポジトリ内のコンテナ内で保存されたすべてのファイルに適用されます．いくら `Devcontainer` が便利で全員に強制させたいからと言って，簡単な操作をするためだけに一度コンテナに入らなければいけないのはセンスがないです．

[non-root-user]: https://code.visualstudio.com/remote/advancedcontainers/add-nonroot-user

[code.visualstudio.com][non-root-user] でも紹介されています．非 `root` ユーザーを作りましょう．

```dockerfile
ARG USERNAME=user-name-goes-here
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Create the user
RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME \
    #
    # [Optional] Add sudo support. Omit if you don't need to install software after connecting.
    && apt-get update \
    && apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

# ********************************************************
# * Anything else you want to do like clean up goes here *
# ********************************************************

# [Optional] Set the default user. Omit if you want to keep the default as root.
USER $USERNAME
```

## 必ずしも Devcontainer で固定する必要はない

次は別の視点から見ます．皆さんにとって開発中のバージョン管理，特にライブラリのバージョン管理はどのようなものを使いますか？という問題です．Python であれば `pip` を使うでしょうし，Golang であれば `go.mod` や `go.sum` などのバージョンの固定を行う仕組みを使うでしょう．それらも確かに，バージョンを固定するための仕組みであり，機能しています．

### アンチパターン: ライブラリのダウンロードを行う（行うな）

それらに対して，「プロジェクトに参加する開発者全員の開発環境を一定に揃える」ために Devcontainer でさらにバージョン固定を行うことは明らかな過剰剛性であり，センスがありません．当たり前ですが，コンテナは多くのリソースを使用します．特にこの過剰剛性によってイメージサイズが大きくなることは避けることができません（以前，機械学習用の Devcontainer でこのミスを行いました，どうなったかは想像に難しくありませんが，ビルド時間は長くイメージサイズの大きなコンテナができました）．さらに，概してライブラリというのは変更や追加されることなど開発用のツールほど少なくはありません．ライブラリの変更が行われるたびにコンテナのビルドが行われては開発どころではないでしょう．

---

ここで，2個の問題が出てきます．一つ目はコンテナのビルドでライブラリやパッケージの取得を行わない場合，コンテナとしてはライブラリもパッケージも含まれていないわけですが， Devcontainer 起動毎にどこかで追加する必要がある，という問題が，二つ目はどこまでをコンテナのビルドで含めるべきか，という問題です．

[lifecycle]: https://containers.dev/implementors/json_reference/#lifecycle-scripts
一つ目の問題は，幸いなことに `devcontainer.json` の [lifecycle] 周りの設定によって解決することができます．

- `postCreateCommand`: コンテナのビルドが完了した後に実行されます．
- `postStartCommand`: コンテナの実行が開始した後に実行されます．
- `postAttachCommand`: コンテナへのアタッチが完了した後に実行されます．

![](https://storage.googleapis.com/zenn-user-upload/ea7f7fc8b456-20221119.png)
*いずれの場合もボリュームがマウントされ，非rootユーザで実行されていることを示す図*

これらは Devcontainer の実行開始（再ビルド後含む）毎に実行されるので，このタイミングで `pip install` や `go mod download` などを実行することができます．複数コマンド実行する場合は一度シェルスクリプトにまとめてスクリプトの実行を指定するのが楽です．

二つ目の問題について，ケースによって色々ありますが，主に私が判断材料としている点は以下の三点です．

- 前述したような過剰剛性となる
- バイナリが大きいが，キャッシュを用いてビルドなどを高速化できる場合
- チーム内の開発用ツールで更新頻度が高いなど，コンテナでイメージとして管理するほうが面倒である

### 積極的にキャッシュを使う

先ほどの続きですが，キャッシュが使える場面であれば積極的に使っていくべきです．様々な言語やツールでキャッシュやリポジトリごとにライブラリを管理する仕組みがあると思うのでそれを使っていきましょう．一番簡単な方法は `node_modules` や `venv` など，リポジトリのあるディレクトリ内にキャッシュを持ってしまう方法です．これらはコマンド実行した時にはボリュームとしてマウントされますし，ディレクトリさえ消せばキャッシュも削除できるので効率的です．

キャッシュとして `docker compose` のボリュームを使う場合は，マウントしたディレクトリに対して非ルートユーザにアクセス権限があるかどうかを考慮してください（後述しますが，できない場合は `go mod download` などと同じタイミングで権限を書き換える方法が便利であると思われます）．

## ホストマシンを気にかける

[^2]: https://github.com/microsoft/WSL/issues/4739

さて，先ほど「Devcontainer を用意する意義」の章で「どのような環境のマシンを使用しているかに関わらず」と言いましたが，現実問題として，そんなに甘くないというのが所感です．CPU アーキテクチャ（特に多いのが x86_64 と arm64）によるイメージの差異は回避することができませんし， Windows と Linux でファイルシステムが違うために，バインドマウントを行った際に不都合が生じる（私の衝突した問題としては，ファイルが保存されたことを通知する OS 由来のイベントが発火しなかった[^2]）などの問題があります．このうち，前者はまだ各個人で改善の余地があるので見ていきたいと思います．

この問題を解決する方法は3種類あります．

一つ目は， Dockerfile をマルチアーキテクチャに対応する方法です．この方法ではより多くのアーキテクチャに対して自然に対応できますが，そもそも使用するツールによってはソースコードからビルドする必要があったり，そもそもビルドする手段がなかったり，不可能を含めて実現困難である点や Dockerfile の記述でかなり気を遣わなければいけない点にあります．
二つ目に，コンテナとして実行するアーキテクチャを統一する代わりに異なるアーキテクチャを実行できるマシンを用意する方法です．この方法では，一つのアーキテクチャに統一されるので Dockerfile も比較的単純になりますが，代わりに間に異なるアーキテクチャとして実行するマシンにとっては低速になりますし，異なるアーキテクチャの挙動をどこまで再現できるか，という点も気にすることになります．（これをテストする環境が私の手元にないので手放しに良いとは言えない，というのが執筆者の本音です）
三つ目に，そもそもマルチアーキテクチャを諦めて開発に使用するコンピュータの CPU アーキテクチャを統一する方法です．これは一番単純な方法ですべてが丸く収まりますが，マルチアーキテクチャは諦めることになります．最近までは `x86_64` が大半だった（偏見）のでこの方法でも問題にすらならなかったのですが， Apple Silicon が出てきたことで状況が変わりました．

どの方法も一長一短なので，どの方法で解決するかは人それぞれです．

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

基本的にベースは `docker-compose.yml` ベースで作成するテンプレートのものを流用する形となっています． `customizations` ブロックで Devcontainer 中に使用する拡張機能や適用する VSCode の設定を行います．
`postAttachCommand` で後述するシェルスクリプトを実行します．ここで `go mod download` などをおこないます．

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

Dockerfile です．Debian にこだわる理由もないし， `apt-get` を呼べばとりあえず必要なものが揃いそう（個人の感想です） Ubuntu をベースにしています．正味，なれたディストリビューションでいい気がします．非rootユーザをここで作ります．

ダウンロードしたファイルを基本的にパイプに通して解凍するなり実行するなりしているのは私の趣味の問題なので気にしないでください．本当はインストールが完了したら，インストールに使用したファイルも消すべきなんですが完全に忘れています．また，過去に自分で書いたコードから流用している部分もあるので Golang のインストールなど CPU アーキテクチャに合わせたインストール方法になってるものもあります（これでもまだきれいな方なんです）．

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

docker-compose.yml です． `command` が Devcontainer のテンプレート通りのコマンドで，これによってコンテナが永続的に稼働します．また， `volumes` の `..:/workspace:cached` もテンプレート通りのバインドマウントで，これでワークスペースディレクトリをコンテナにボリュームマウントを行います． `gohome` は文字通り， Golang のホームディレクトリを丸ごとボリュームとしたものでパッケージやインストールバイナリのキャッシュとして機能します．

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

最後に Devcontainer 起動毎に実行されるシェルスクリプトです．かなり雑に組まれてはいるのですがはじめにワークスペースディレクトリに移動します．次に， `chown` でホームディレクトリの所有者を設定しています．これは一見何の意味もありませんが， `docker-compose.yml` で設定された `gohome` ボリュームに対して機能します．既定でボリュームは root ユーザしかアクセスすることができないので，これがないとおそらくそのあとの `go install` などを行った際に `Permission Denied` で失敗します．最後に `VSCode` の Golang 用の拡張機能で使用するようなツールをインストールして，必要なパッケージをダウンロードします．

---

このように，気をつけなければいけないことが山とある Devcontainer ですが，すべて差し引いたとしても，再現性のある開発環境，あるいはそれ自体が開発環境構築のドキュメントとして機能するというメリットは長期的であればあるほど，大規模であればあるほど有効に機能していくと私は考えます．
失敗談ベースにこの記事を整理したので至らない点が多くあったかと思いますが，とりあえず今回はこれで締めさせて頂きます．ありがとうございました．

# P.S.

これ全く関係ない話なんですが，正直な話するとコンテナ（特に Devcontainer）をしっかりやるまで，ロクに Linux コマンド使っていなかったんですよね．
はじめての Linux ，ぜひ Devcontainer からはじめてみるのはどうでしょう？
