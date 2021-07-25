---
title: 【ツールを整理してみた】ソフトウェア編
---
[「普段使っているツールを整理して何に使っているのかを再認識してみた」シリーズ](https://zenn.dev/streamwest1629/books/introduce_devtools) 第一回となる今回は、広い意味でアプリケーション開発を行う際に使用しているツールを紹介していきたいと思います。
言い方を変えると、特定の言語でしか使用しない、特定の目的でしか使用しないようなツールではなく、おそらく割と広い範囲で使うだろうと思ったものを今回は紹介していきます。
今回はこの後に続くシリーズの概論的な立ち位置の記事にしようと思っています。

# Visual Studio Code(VSCode)
皆様もお馴染みのソースコードエディタです。GUIの使える環境においてという条件付きではありますが、Vim勢を確実に滅ぼしていく救世主であり、拡張機能をうまく利用したり新しく制作したりすることでどのような場面でも対応することができる優れものです。

これについてはすべてを語りつくそうと思うとそれだけで日をまたいでしまうほど色々な機能があるため [VSCodeの拡張機能について語る回](vscode-ext)、[VSCodeの設定やショートカットについて語る回](vscode-config)を次回以降に設けています。


> **VSCode** [Visual Studio Code -  Code Editing Redefined.](https://code.visualstudio.com)


# Git(Github)
これも皆様お馴染みのコードの管理ツールです。ディレクトリを一つのリポジトリという単位で扱い、その中で発生した変更などを記録していくことで修正を戻したりすることができます。

[Githubについて語る回](github)を次回以降に設けています。


> **Git** [Git](https://git-scm.com)
**Github** [Github: Where the world builds software](https://github.com)

## Git bash
Gitをダウンロードした際に付属してくるソフトウェアです。Windows上でシェルスクリプトを実行したりコマンドを実行するのに使用します。VSCodeのターミナルの初期設定ではPowershellとなっていますが、Git bashにしておいた方が後々楽です。

## Github Desktop
通常CUIツールのGitですが、それをGUIツールとして管理しやすくしたものがGithub Desktopです。その名前からまるでGithub内のリポジトリにしか使えないような名前をしていますが、実際にはローカルや別サーバーのリポジトリ（多分）にも対応しているので体感としてはGitがそのままGUIツールになった感じで扱えます。


> **Github Desktop** [Github Desktop | Simple collaboration from your desktop](https://desktop.github.com/)

# npm
本来は `node.js(javascript)` に同梱しているパッケージ管理ツールです。ですがnpmから配布されているパッケージを組み合わせながらスクリプトを書くことで開発を高速化させることができます。
[パッケージについて語る回](npm-pkg)を次回以降に行います。

> **npm (Node.js同梱)** [Node.js](https://nodejs.org/en/)

# MSYS2
パッケージ管理ツールです。Windowsでgcc(g++)を使用するために使用します。他にもインストーラーとして配布されていないソフトウェア（特にOSS）を管理するためにとても便利なツールです。
その昔MSYSというパッケージ管理ツールがあって、その後継ということでMSYS2らしいです（知らんけど）。

> **MSYS2** [MSYS2](https://www.msys2.org)
