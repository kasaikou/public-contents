---
title: Go言語で(関数|メソッド|変数)がパッケージ外部に公開されるかどうかを見分けるチートシート
emoji: 😎
type: tech
topics: [golang, パッケージ, チートシート, インターフェイス, 埋め込み]
published: true
---

# Gopherの皆様、いかがお過ごしでしょうか。
今回はGo言語のパッケージ単位で見た時に公開されるかどうかを見極めるチートシートを書きました。この記事のターゲット層は「とりあえずGo言語を触ってある程度の文法を理解しました」くらいの人を対象にしてみましたが、後半はそれなりに読み応えのある文章にしたつもりなのでつよつよの方のご意見も伺えたらと思います。

# 前提知識
## パッケージ `package` 
Go言語は各ディレクトリごとに **パッケージ** という単位で分割されます。このパッケージ内部ではローカル変数を除けばどの変数へも等しくアクセスできます。
言い方を変えれば、 **「同一ディレクトリ上で定義されたオブジェクトにアクセスすることができる」** というわけです。
:::message
この時にサブディレクトリはまったく関係のない別モジュールとして扱われることに注意してください。
:::
各パッケージ内のファイルの先頭には、
```go
package rehearsal
```
という風にパッケージ名を明記します。これは同一ディレクトリ内のファイルで統一（例外あり）しなければいけません。実行ファイルとなる（`func main(){}` のある）パッケージには `main` としなければいけません。それ以外は原則としてディレクトリ名にしておくのが一般的です。理由は後述します。
:::message
例外としてテスト用に `<package-name>_test` というパッケージを設定できます。
:::

## パッケージのインポート `import` およびその参照
Go言語は各パッケージをインポートという形でインポートします。たとえば、[`github.com/gin-gonic/gin`](https://pkg.go.dev/github.com/gin-gonic/gin) というGitHub上で公開されているパッケージをインポートするときは
```go
package main

import (
    "github.com/gin-gonic/gin"
)
```
という風に記述することでインポートを定義します。
:::message
実際には `go mod` 周りの設定があります。 GitHubのリポジトリとして管理している場合は、Bashかコマンドプロンプト、シェルスクリプトを開いてそのリポジトリの最上位ディレクトリに移動して以下の二個のコマンドを実行してください。
```
go mod init github.com/<user-name>/<repository-name>
go mod tidy
```
とく二行目の方は外部のパッケージ（この時自分のリポジトリ内のディレクトリはカウントしなくてよい）をインポートするたびに実行してください。
:::

外部パッケージ内のオブジェクトにアクセスするときは `<package-name>.<object>`という風に定義します。
以下は上で使用した `github.com/gin-gonic/gin` の [Quick start](https://pkg.go.dev/github.com/gin-gonic/gin#readme-quick-start) です。
```go
package main

import "github.com/gin-gonic/gin"

func main() {
	r := gin.Default() // 今回はここが一番大事
	r.GET("/ping", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "pong",
		})
	})
	r.Run() // listen and serve on 0.0.0.0:8080 (for windows "localhost:8080")
}
```
このパッケージは `gin` という名前のパッケージで、6行目で `gin.Default()` という関数を呼び出しているのがわかります。
この時、パッケージ名をディレクトリ名にしておくとわかりやすい、というのが前述したパッケージ名をディレクトリ名にしておく理由です。

## 公開オブジェクトと内部オブジェクト
**Go言語では他の言語のクラスにある `private` `public` のような概念は存在しません。** 代わりにパッケージの外部からアクセスできるオブジェクトとそうでないオブジェクトの二種類を定義できます。この二種類は **頭文字が大文字かどうかで** 決定します。
:::message alert
外部からアクセスできるオブジェクトとできないオブジェクトのそれぞれの名前がわからないので、この記事では暫定的に「公開オブジェクト」と、「内部オブジェクト」と呼称します。
つよつよの人助けて。
:::
```
github.com/atamanowarui/ripojitori
 ┣ dir
 ┃  ┗ sub.go
 ┗ main.go
```
具体例を考えましょう。まず初めに上のようなファイル構造を考えます。 **ここで、`main.go` と `sub.go` はそれぞれ別のパッケージである** ということがわかっていれば他はどうでもいいです。
次に `sub.go` で以下のような変数を定義します。
```go
package dir

var (
    Foo int     // 頭文字が大文字だから公開オブジェクト
    Bar string  // 頭文字が大文字だから公開オブジェクト
    foo int     // 頭文字が小文字だから内部オブジェクト
    bar string  // 頭文字が小文字だから内部オブジェクト
)
```
次に `main.go` でこれらの変数を呼び出します。
```go
package main

import (
    "github.com/atamanowarui/repositori/dir" // dirパッケージをインポート
)

func main() {
    print(dir.Foo)  // 問題なし
    print(dir.Bar)  // 問題なし
    pring(dir.foo)  // これダメです。そんな変数は定義されていないことになっています。
    pring(dir.bar)  // これダメです。そんな変数は定義されていないことになっています。
}
```

という風に、 **小文字のオブジェクトは外部に公開されない** というルールがあります。
しかし、これをそれなりにうまく利用することでおもしろいことができる、ということに気づいたので今回は記事にしてみた次第です。

:::message
以降は `sub.go` の `package dir` 以外ところだけ書いていきます。
:::

# イージーモード
はじめはイージーモードです。前述した内容も踏まえながら進めていきましょう。これ以降もこういう書き方をします。
## Season 1 `Globals`
はじめはC++でいうところの「グローバルな」奴らです。
```go
const (
    Bar = "neko"    // 1
    bar = "tabetai" // 2
)
var (
    Foo = 10        // 3
    foo = int64(10) // 4
)
func Function() {}  // 5
func function() {}  // 6
```
:::details 答えはここから
| 番号 | オブジェクト | オブジェクトについて | 外部から参照できるか |
| :--: | :--- | :--- | :--: |
| 1 | **`Bar`** | 公開された定数 | **OK** |
| 2 | **`bar`** | 内部の定数 | **NG** |
| 3 | **`Foo`** | 公開された変数 | **OK** |
| 4 | **`foo`** | 内部の変数 | **NG** |
| 5 | **`Function`** | 公開された関数 | **OK** |
| 6 | **`function`** | 内部された関数 | **NG** |
:::
:::details 解説
これはおそらく想像通りだったと思います。何せイージーモードですからね。
:::
## Season 2 `Types`
続いて、型の定義です。Go言語では「構造体」「インターフェイス」「別名定義」の三種類があります。それぞれ見ていきましょう。
```go
type (
    Msg struct {        // 1
        Member string   // 2
        member string   // 3
    }
    msg struct {        // 4
        Member string   // 5
        member string   // 6
    }
    Sender interface{}  // 7
    sender interface{}  // 8
    Identifier string   // 9
    identifier int64    // 10
)
```
:::details 答えはここから
| 番号 | オブジェクト | オブジェクトについて | 外部から参照できるか |
| :--: | :--- | :--- | :--: |
| 1 | **`Msg`** | 公開された構造体 | **OK** |
| 2 | **`Msg.Member`** | 公開された構造体の公開メンバー変数 | **OK** |
| 3 | **`Msg.member`** | 公開された構造体の内部メンバー変数 | **NG** |
| 4 | **`msg`** | 内部の構造体 | **NG** |
| 5 | **`msg.Member`** | 内部の構造体の公開メンバー変数 | **NG** |
| 6 | **`msg.member`** | 内部の構造体の内部メンバー変数 | **NG** |
| 7 | **`Sender`** | 公開されたインターフェイス | **OK** |
| 8 | **`sender`** | 内部のインターフェイス | **NG** |
| 9 | **`Identifier`** | 公開された別名定義 | **OK** |
| 10 | **`identifier`** | 内部の別名定義 | **NG** |
:::
:::details 解説
まあこれも想定の範囲内かと思います。現状では公開されていない構造体の変数は頭文字が大文字かどうかにかかわらず、そもそも構造体を参照できないので参照できません。
まあ、この文言でお察しの方は多いと思いますが、参照方法は存在するのでちゃんと分別をつけておきましょう。
:::
## Season 3 `Member functions`
続いて、メンバー関数です。Go言語は「構造体」「別名定義」のそれぞれにメンバー関数を与えることができます。
:::message
この **「別名定義」にメンバー関数を与えられる** ということを意外と知られていない気がします。かなり有用ですのでこの際に覚えて帰ってください。
:::
```go
type (
    PStruct struct{}    // PublicStructが長かったので
    iStruct struct{}    // internalStructが長かったので
    PDefine string
    iDefine string
)

func (p *PStruct) Get() {}  // 1
func (p *PStruct) get() {}  // 2
func (i *iStruct) Get() {}  // 3
func (i *iStruct) get() {}  // 4
func (p *PDefine) Get() {}  // 5
func (p *PDefine) get() {}  // 6
func (i *iDefine) Get() {}  // 7
func (i *iDefine) get() {}  // 8
```

:::message alert
[ご指摘](https://zenn.dev/link/comments/a68db553f51142)をいただいて、ミスがあった事を確認したので回答を修正しています。
申し訳ございません。
:::

:::details 答えはここから
| 番号 | オブジェクト | オブジェクトについて | 外部から参照できるか |
| :--: | :--- | :--- | :--: |
| 1 | **`(*PStruct).Get()`** | 公開された構造体の公開メンバー関数 | **OK** |
| 2 | **`(*PStruct).get()`** | 公開された構造体の内部メンバー関数 | **NG** |
| 3 | **`(*iStruct).Get()`** | 内部の構造体の公開メンバー関数 | **NG** |
| 4 | **`(*iStruct).get()`** | 内部の構造体の内部メンバー関数 | **NG** |
| 5 | **`(PDefine).Get()`** | 公開された別名定義の公開メンバー関数 | **OK** |
| 6 | **`(PDefine).get()`** | 公開された別名定義の内部メンバー関数 | **NG** |
| 7 | **`(iDefine).Get()`** | 内部の別名定義の公開メンバー関数 | **NG** |
| 8 | **`(iDefine).get()`** | 内部の別名定義の内部メンバー関数 | **NG** |
:::
:::details 解説
Season 2の解説とほぼ同じです。内部の構造体にアクセスすることができないので、その中にある公開メンバー関数へアクセスすることはできません。しかし、いくつかの方法でこれを公開させることができるのでその方法を次で紹介します。
:::
# 中級（この記事のメインテーマ）
ここでは、多少気持ちの悪いアクセス方法を用いて今まで公開できなかった方法を用いて主にメンバー変数をこじ開けていきたいと思います。
## インターフェイス `Interface Functions`
はじめに、インターフェイスを用いて内部のオブジェクトにある公開メンバー関数をこじ開けていきたいと思います。 **インターフェイスとして定義できる関数は常に外部に公開されなければいけない** という規則が存在します。ですので今回は内部関数を考えないことにします。
```go
type (
    Interface interface {
        Get()                   // 1
    }
    internal struct {}
)

func (i *internal) Get() {}     // 1, 2
func (i *internal) Quit() {}    // 3
```
当たり前ですが、internalを直接参照することはできません。しかし、Interfaceを経由させることで、Interfaceで定義している `Get()` を呼び出すことができます。よって表は以下のようになります。
| 番号 | オブジェクト | オブジェクトについて | 外部から参照できるか |
| :--: | :--- | :--- | :--: |
| 1 | **`Interface((*internal)).Get()`** | 公開インターフェイスとして参照された内部の構造体の公開関数 | **OK** |
| 2 | **`(*internal).Get()`** | 内部の構造体の公開関数 | **NG** |
| 3 | **`(*internal).Quit()`** | 内部の構造体の公開関数 | **NG** |

## 構造体の埋め込み `Embedded Structs`
Go言語にはオブジェクト指向のような継承の概念がありません。その代わりに構造体の埋め込みというを用いることができます。
```go
type (
    embedded struct {
        Foo int             // 1
        foo string          // 2
    }
    Embedding struct { *embedded }
)

func (e embedded) Bar() {}  // 3
func (e embedded) bar() {}  // 4
```
構造体の埋め込みでは、埋め込まれた構造体のメンバーを埋め込んだ構造体のメンバーとして扱うことができるため、表は以下の通りになります。
| 番号 | オブジェクト | オブジェクトについて | 外部から参照ができるか |
| :--: | :--- | :--- | :--: |
| 1 | **`(*Embedding).Foo`** | 公開構造体に埋め込まれた内部構造体の公開メンバー変数 | **OK** |
| 2 | **`(*Embedding).foo`** | 公開構造体に埋め込まれた内部構造体の内部メンバー変数 | **NG** |
| 3 | **`(*Embedding).Bar()`** | 公開構造体に埋め込まれた内部構造体の公開メンバー関数 | **OK** |
| 4 | **`(*Embedding).bar()`** | 公開構造体に埋め込まれた内部構造体の内部メンバー関数 | **NG** |
最後に、結構難しめの問題を解いてこの記事を終わりにしたいと思います。

# 上級
ここでは、実際に遭遇したケース（かなりのレアケースだと信じたい）を参考に合体させた奴を作りました。もう正誤を記述するのは大変なので、外部から呼び出せるケースだけ回答に載せておきます。
呼び出せる構造体とインターフェイスの組み合わせがいくつあるか、そしてその呼び出し方法が正しいかどうかを自分で検証してみてください。
```go
type (
    Call interface {
        Change(args []string)
        Quit()
        Get() []Part  // この関数でPartが取得可能
    }
    Executer interface {
        SetValue(idx int, val string) error
        Execute() Call  // この関数でCallが取得可能
    }
    Part interface {    // 関係ないけど標準パッケージのio.ReadCloserと同等
        Reader(dst []byte) (int, error)
        Close() error
    }
    embedded struct {}
    embeding struct { *embedded }
)

func MakeExecuter() Executer { return &embedded } // この関数でExecuterが取得可能

func (i *embedded) Change(args []string) {}
func (i *embedded) Quit() {}
func (i *embedded) Get() []Part { return &embedding{ i } }
func (i *embedded) SetValue(idx int, val string) {}
func (i *embedded) Execute() Call { return i }
func (i *embedded) Close() error { return nil }
func (i *embeding) Reader(dst []byte) (int, error) { /* 省略 */ }
```
:::details ヒント（外部から呼び出せる組み合わせの数）
外部から呼び出せる組み合わせの数は7個です。
:::
:::details 答えはここから
| 番号 | オブジェクト | 解説 |
| :--: | :--- | :--- | :--- |
| 1 | **`Call((*embedded)).Change()`** | `Call` インターフェイスとして呼び出した `embedded` 構造体の公開関数。 |
| 2 | **`Call((*embedded)).Quit()`** | `Call` インターフェイスとして呼び出した `embedded` 構造体の公開関数。 |
| 3 | **`Call((*embedded)).ViewState()`** | `Call` インターフェイスとして呼び出した `embedded` 構造体の公開関数。 |
| 4 | **`Executer((*embedded)).SetExecute()`** | `Executer` インターフェイスとして呼び出した `embedded` 構造体の公開関数。 |
| 5 | **`Executer((*embedded)).Execute()`** | `Executer` インターフェイスとして呼び出した `embedded` 構造体の公開関数。 |
| 6 | **`Part((*embeding)).Reader()`** | `Part` インターフェイスとして呼び出した `embeding` 構造体の公開関数。 |
| 7 | **`Part((*embeding)).Close()`** | `Part` インターフェイスとして呼び出した `embeding` 構造体に埋め込まれた `embedded` 構造体の公開関数。 |
:::

いかがだったでしょうか。パッケージ間でアクセス制限を設けたり、インターフェイスで内部の構造体を返したりすることで可読性の高いソースコードや汎用性の高い割に安全性も担保されたコードが完成します。これを機に快適なGopherライフを送りましょう！

# 最後に自己紹介
大学生してます。Go言語を用いたプロダクト「rehearsal」を片手に持って技育展2021に出展します。今欲しいものは応援とTwitterのフォロワーです。

@[card](https://twitter.com/streamwest1629)
