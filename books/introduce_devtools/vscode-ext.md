---
title: 【ツールを整理してみた】VSCode拡張機能編
---
[「普段使っているツールを整理して何に使っているのかを再認識してみた」シリーズ](https://zenn.dev/streamwest1629/books/introduce_devtools) 第二回となる今回は、普段ボクが使用しているVSCodeの拡張機能を紹介していきたいと思います。
VSCodeの話題はあまりにも大きいので前半と後半に分けて話していきます。今回は前半として拡張機能でつよつよコードエディタにします。後半でVSCode本来の機能の有能な機能の設定方法やショートカットについて語る事にしています。
先に拡張機能を設定する方針にしました。理由は二つあって、一応何も設定しなくてもVSCodeそれ自体は動くので設定を触るのは後回しでもいいと思ったのが一つ目の理由です。けれど、それ以上に二つ目の理由が大事で、設定を触ったりショートカットキーを覚えるよりも、拡張機能を導入する方が設定を触るよりもはるかに簡単だからその導入を先にすることにしました。

**それでは始めていきましょう！テンポよくいきます！**

# 言語サポート系
これは拡張機能というべきなのか何なのかよくわかりませんが、VSCodeがコードエディタたるために、この手の拡張機能を導入することで全ての作業を簡単にします。ここでいう言語サポートはデバッグ機能などを提供していたりコードの可読性を向上させるツールということで認識しておいてください。

これは皆さんが使っている言語によって必要なものが変わってしまうので一応ボクが導入して、かつ割と使っている物を紹介していきます。
ここでは簡単な紹介だけしてより詳細な紹介はそれぞれの言語に特化した記事でまとめていきたいと思います（分類が面倒なのでアルファベット順になっていると思います）。

- **C/C++** C/C++の言語サポート。使用するコンパイラや標準ライブラリの指定など、かなり細かいところまで設定ができる。 [導入する](vscode:extension/ms-vscode.cpptools)
  - **C++ Intellisense** C/C++の入力支援機能 [導入する](vscode:extension/austin.code-gnu-global)
- **CMake** CMakeの言語サポート。 [導入する](vscode:extension/twxs.cmake)
- **Dart** Dartの言語サポート。 [導入する](vscode:extension/dart-code.dart-code)
- **Docker** Dockerの言語サポート。 [導入する](vscode:extension/ms-azuretools.vscode-docker)
- **Flutter** Flutterの言語サポート。ホットリロードに対応している。 [導入する](vscode:extension/dart-code.flutter)
- **Go(Golang)** Go言語(GoLang)の言語サポート。 [導入する](vscode:extension/golang.go)
- **HTML CSS Spport** HTML/CSSの言語サポート。 [導入する](vscode:extension/ecmel.vscode-html-css)
  - **IntelliSense for CSS class names in HTML** CSS名を用いたHTMLの入力支援機能 [導入する](vscode:extension/zignd.html-css-class-completion)
- **Jupyter** Jupyter notebookの言語（？）サポート。 [導入する](vscode:extension/ms-toolsai.jupyter)
- **Markdown All in One** Markdownの言語（？）サポート。プレビューや目次作成などの機能もある。 [導入する](vscode:extension/yzhang.markdown-all-in-one)
- **npm** npmの言語（？）サポート。[導入する](vscode:extension/eg2.vscode-npm-script)
  - **npm Intellisense** npmの入力支援機能。 [導入する](vscode:extension/christian-kohler.npm-intellisense)
- **Python** Pythonの言語サポート。 [導入する](vscode:extension/ms-python.python)
  - **Pylance** Pythonの型チェックツール。 [導入する](vscode:extension/ms-python.vscode-pylance)
- **Sass** sassの言語サポート。 [導入する](vscode:extension/syler.sass-indented)
- **Swagger Viewer** Swaggerのビューワ。 [導入する](vscode:extension/cssho.vscode-svgviewer)
- **YAML** yamlの言語（？）サポート。 [導入する](vscode:extension/redhat.vscode-yaml)

# Git連携系
Gitと連携することでつよつよになったり楽しくなれる拡張機能です。
言語サポートほど種類は多くはなく、次回以降のGit回はGitのメイン機能に焦点を置きたいのでこれはできる限り詳しく書いていきたいと思います。

## GitLens
VSCode上でGitの基本的な操作(add, commit, pushなどなど)を行うための拡張機能です。[導入する](vscode:extension/eamodio.gitlens)

## Git Graph
左下に出てくる `Git Graph` のボタンが出てきて、それを押すとGitのコミットやブランチの進捗の様子を可視化します。これを見るとチーム開発がより一層楽しくなります。[導入する](vscode:extension/mhutchie.git-graph)

# チーム開発系
次はGitを使う/使わないにかかわらず、チーム開発で有用そうなものを紹介します。

## EditorConfig for VS Code
ソフトタブとハードタブなどの、各エディタごとに異なる設定をディレクトリ内で統一します。複数のコードエディタ間で設定を共有することができます。
ちなみに、VS Codeだけであれば `.vscode/setting.json` で詳細な設定を行うことができます。[導入する](vscode:extension/editorconfig.editorconfig)

## Live Share
リアルタイムな共同編集を実現します。[導入する](vscode:extension/ms-vsliveshare.vsliveshare)

# 開発支援系
実際にソースコードを記述していく際にかなり有用となるような拡張機能です。これは紹介したい順に並べて紹介します。

## vscode-icons
VSCodeのファイルエクスプローラに表示されるアイコンの種類を一挙に増やします。多くのプログラミング言語に対応している上に、特定のディレクトリ名のディレクトリのアイコンも変えるので (例: `src` `api` `img` `venv` など) ファイルエクスプローラの視認性が上がります。[導入する](vscode:extension/vscode-icons-team.vscode-icons)

## Visual Studio IntelliCode
複数の言語に対応したAI支援の入力支援拡張です。Visual Studioが持つ入力支援機能と同等のものを得られる（んじゃないかな？） [導入する](vscode:extension/visualstudioexptteam.vscodeintellicode)

## Better Comments
`// todo:` や `// ?` などに反応してそのコメントを強調表示させることができます。修正箇所や疑問点などをわかりやすく強調表示されます。 [導入する](vscode:extension/aaron-bond.better-comments)

## Pretter - Code formatter
コードフォーマッターです。保存したときに可読性の高いコードに仕上げることができます。 [導入する](vscode:extension/esbenp.prettier-vscode)

## Color Highlight
`#ff0000` や `0xff0` という表記に反応してそのカラーコードに対応した色のハイライトを付加します。 [導入する](vscode:extension/naumovs.color-highlight)

## Todo+
ソースコード上の `// todo: ` を検索したサイドバーに一覧として表示します。[導入する](vscode:extension/fabiospampinato.vscode-todo-plus)

## Character Count
Markdown の文字数カウンタです。[導入する](vscode:extension/stevensona.character-count)
