---
title: ローカル開発環境とコンテナの話前編 - プロジェクトとデータセット
---

# プロジェクトとコンテナ
前回まで，ローカル環境とコンテナをつなぐための方法について簡単に確認してきました．この回からは第六回までで作ったコンテナに対して，色々なものを追加していきたいと思います．前回，前々回では触れなかったGPUの話もここでできたらと思います．

# プロジェクトをバインドマウントで繋ごう
とはいえ，今回することは極端むずかしい訳ではなく，プロジェクトを格納しているローカルディレクトリを前回使用したバインドマウントで繋ぐという，それだけです．

今回は`docker-compose.yml`と同じ階層に`projects`ディレクトリを作成し，これを`/workspace`ディレクトリとしてコンテナにマウントします．
ローカルから見ると以下のようなファイル構成ですね．
```md
- .docker
    - Dockerfile
    - requirements.txt
- projects
    - <your projects...>
- docker-compose.yml
- .gitignore
```

前回までに大体の解説は行ったので結論だけ見ていくと，docker-compose.yamlの中にある `jupyterlab` サービスに`volumes`を追加します．

以下の通りになります．
```yaml
jupyterlab:
  build:
    context: .docker
  command: python3 -m jupyter lab --ip=0.0.0.0 --port 8888 --allow-root
  ports:
    - 8888:8888
  volumes:
    - ./projects:/workspace
```

2行書き足すだけだから難しいことは何もないですね．

# データセットをバインドマウントで繋ごう
問題はここからです．データセットをバインドマウントで繋ぐのですが，データセットはプロジェクトに比べてファイルが大きくなりがちなので，ドライブを分けなければいけなかったり，あるいはもうすでにファイルが決まったドライブやディレクトリの中にあり，中々動かせないなどの制約があったりします．

そこで，前々回の`.env`ファイルでローカル環境に依存する部分を書くことにしましょう．

## 1. `.gitignore`ファイルに`.env`ファイルを追加する
言葉の通りです．以下の一文をdocker-compose.ymlと同階層にあるgitignore`に追加しましょう．
```
.env
```

これで，`.env`ファイルがGitの管理から外れるので，Githubにあがることはなくなりました．所謂「Githubにあげる必要のないもの，あげてはいけないもの」を`.env`ファイルに書くことにします．

色々な方が書いているのですが，この場合，.gitignoreは同階層かその下の階層にある`.env`ファイル（`*.env`ではない）をGitの管理から外すことを意味します．

## 2. `.env`ファイルにディレクトリパスを指定する
`docker-compose.yml`と同階層の`.env`ファイルにデータセットが置いてあるディレクトリへのローカルパスを書きます．以下のような感じで書いてください．これは私の例です．
```env
DATASET_DIR=D:/mega/datasets
```

左辺の`DATASET_DIR`は`docker-compose.ymlで値を呼び出す際に用いるキーで右辺がそのキーです．
余談ですが，私は[mega.nz](https://mega.nz)のWindows用ソフト「MEGAAsync」のファイル同期機能を使って複数マシンでデータセットを共有しています（現時点の方法，今後S3に乗り換える可能性がかなり高い）．

## 3. `.env`ディレクトリのキーを`docker-compose.yml`で使用する
では最後です．最初に書いた`volumes`に値を追加して完成です．
```yaml
jupyterlab:
  build:
    context: .docker
  command: python3 -m jupyter lab --ip=0.0.0.0 --port 8888 --allow-root
  ports:
    - 8888:8888
  volumes:
    - ./projects:/workspace
    - ${DATASET_DIR}:/dataset
```

これで完成です．このコンテナを実行すると，ローカルのprojectsディレクトリは`/workspace`ディレクトリに，ローカルのデータセットを保管しているディレクトリは **「厳密に** `/dataset` ディレクトリにマウントされました．
これによって，デバイスを変えてデータセットディレクトリが移動したとしても，`.env`ディレクトリを書き換えるだけで`/dataset`ディレクトリにマウントできるようになりました．

次回はGPUをコンテナに繋ぎたいと思います．
