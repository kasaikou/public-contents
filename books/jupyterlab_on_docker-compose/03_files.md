---
title: JupyterLabを動かそう前編 - ファイル構成
---
# JupyterLabを動かそう
はじめに，使い物になるかならないかは別問題にして，とりあえずJupyterLabを動かす所までを見ていきたいと思います．

# ファイル構成について
結論から言うと，この（中編後編を合わせた）章の最終的なファイル構成は以下のような形になります．

```md
- .docker               [dockerコンテナに関わるディレクトリ]
    - Dockerfile        
    - requirements.txt
- docker-compose.yml
- .gitignore (しばらくは空のファイルで問題なし)
```

一応解説すると，基本的にはコンテナに関わるファイルを`.docker`ディレクトリ内に入れています．
`docker-compose.yml`と`./.env`だけ例外的にルートディレクトリに置いているのはDocker (Compose)のコマンドを長くしたくないから（我儘）です．

`.gitignore`を入れているのは後から使っていくのは間違いないけど「変更差分」として残しておきたいので今のうちに「追加」しているというだけです．

ボクにはどこをどう見ても余力はないはずですが，何故かこの本を書くためだけにGitHubリポジトリを作ったのでそれを参考にしていただければと思います．

https://github.com/StreamWest-1629/jupyterlab_on_docker-compose

:::message
動作のチェックはしていますが，基本的にWindowsでpushしているのでコンテナ内部から見ると面倒なことになっていると思います．

`git clone`などせず，自分の手で書いてください．
:::
