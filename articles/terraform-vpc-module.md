---
title: 巷の Terraform Module に違和感を感じたので納得できるものを作ってみた
emoji: 🧪
type: tech
topics: [Terraform, VPC, AWS]
published: false
---

今日は最近 Terraform Module に感じていた使いにくさの理由と、その克服方法について AWS VPC を構築しながら整理していきます。

# 世間で使われる Terraform Module に対する違和感

早速ですが、巷で使われている Terraform Module に対して感じた違和感を挙げていこうと思います。

具体例があるとよりわかりやすいかと思いますので、 [terraform-aws-modules/vpc/aws](https://registry.terraform.io/modules/terraform-aws-modules/vpc/aws/latest) を例に取りながら見ていこうと思います。

:::message
このモジュール自体は昔からあるもので、 リポジトリの Star 数も 2.9K (執筆時時点) とよく使われるモジュールの一つかと思われるのでこっちの方がいいよ、という方もいるかもしれません。
:::

以下にサンプルを示します。

構成としては、3 箇所の各 AZ にパブリックサブネット及びプライベートサブネットを構築するものとなっています。

```terraform
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"

  name = "my-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["ap-northeast-1a", "ap-northeast-1c", "ap-northeast-1d"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]

  tags = {
    Terraform = "true"
    Environment = "dev"
  }
}
```

注目してほしい部分は `private_subnets` と `public_subnets` です。これらのパラメータを元にサブネットを作成するのですが、その際に `azs` を参照しています。これにより、インデックスを気にしながら定義していく必要があるのです。

## 違和感その1: 1リソースを作成するために複数のパラメータを参照する必要があること

複数のパラメータを用いることで複数のリソースが構築される多対多の関係は扱いにくいです。
今回の例でいうと、パブリックサブネットを2個作るために、 `azs` と `public_subnets` を見ていく必要があります。

```mermaid
erDiagram

"(Public) Subnet Resources" |o--o| "public_subnets variable" : "インデックスで参照"
"(Public) Subnet Resources" |o--o| "azs variable" : "インデックスで参照"
"(Private) Subnet Resources" |o--o| "private_subnets variable" : "インデックスで参照"
"(Private) Subnet Resources" |o--o| "azs variable" : "インデックスで参照"

"public_subnets variable" {
  0 10.0.101.0/24
  1 10.0.102.0/24
  2 10.0.103.0/24
}

"private_subnets variable" {
  0 10.0.1.0/24
  1 10.0.2.0/24
  2 10.0.3.0/24
}

"azs variable" {
  0 ap-northeast-1a
  1 ap-northeast-1c
  2 ap-northeast-1d
}
```

本来、1個のリソースに対して複数個のパラメータがあるのがわかりやすいと思っていて、どんな形であれ、**「リソースに対応するパラメータリストがある」**という状態が望ましいはずです。

```mermaid
erDiagram

"Subnet A" {
  is_public false
  cidr 10.0.1.0/24
  az ap-northeast-1a
}

"Subnet B" {
  is_public true
  cidr 10.0.102.0/24
  az ap-northeast-1c
}
```

これを Terraform 的に書くのであれば、

```
<任意のリソース名> = {
  <パラメータ1> = ...
  <パラメータ2> = ...
  ...
}
```

という形にきっとなるでしょう。

## 違和感その2: インデックス管理されているので変更（特に削除）に非常に弱い

先ほどのサンプルに以下のような変更を行います。

```terraform
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"

  name = "my-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["ap-northeast-1a", "ap-northeast-1d"]
  private_subnets = ["10.0.1.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.103.0/24"]
  # 変更前はこうだった
  # azs             = ["ap-northeast-1a", "ap-northeast-1c", "ap-northeast-1d"]
  # private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  # public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]


  tags = {
    Terraform   = "true"
    Environment = "dev"
  }
}
```

違いとしては、 `ap-northeast-1c` にあったパブリックサブネットととプライベートサブネットを1個ずつ削除しました。
理想としては、削除した分のリソース（パブリックサブネット、プライベートサブネット、ルートの関連付けなど）が単純に削除されれば良いのですが、実際には

```
Plan: 4 to add, 1 to change, 9 to destroy.
```

となってしまいます。

これは、 Terraform 内部でのリソースの扱いの問題で、 Terraform ではインデックスを含めた同じリソース名の差分を評価するので、変更前の `aws_subnet.public[1]`と 変更後の `aws_subnet.public[1]` の差分を評価します。そのために、 「`aws_subnet.public[2]` `aws_subnet.private[2]` が削除されて `aws_subnet.public[1]` `aws_subnet.private[1]` の AZ を変更する」という扱いになってしまうためです。

このように、 Terraform でインデックスでナンバリングをベースにリソースを定義すると、厳密にその順序と番号を意識しなければいけなくなるのです。

# とりあえず作ってみた

ここまで、一通りの違和感を話したので、とりあえず [kasaikou/vpc/aws](https://registry.terraform.io/modules/kasaikou/vpc/aws/latest) で作ってみました。

コンセプトとしては以下の通りです。

- **「リソースに対応するパラメータリストがある」という関係にあること**
- **インデックスにナンバリングを使わないこと**

一旦上と同様のコードを書いてみます。

```terraform
module "vpc" {
  source  = "kasaikou/vpc/aws"
  version = "v0.1.1"

  name       = "my-vpc"
  cidr_block = "10.0.0.0/16"

  subnets = {
    "public-primary" = {
      availability_zone = "ap-northeast-1a"
      cidr_block        = "10.0.1.0/24"
      route_tables      = ["igw"]
    }
    "public-secondary" = {
      availability_zone = "ap-northeast-1c"
      cidr_block        = "10.0.3.0/24"
      route_tables      = ["igw"]
    }
    "private-primary" = {
      availability_zone = "ap-northeast-1a"
      cidr_block        = "10.0.101.0/24"
    }
    "private-secondary" = {
      availability_zone = "ap-northeast-1c"
      cidr_block        = "10.0.103.0/24"
    }
  }
}

```

ミソなのはサブネットの定義の仕方です。型的に `map(object())` で定義していて、「リソース名とそのパラメータ」という関係をここで定義しています。これによって違和感その1で取り上げた **「リソースに対応するパラメータリストがある」** を達成することができます。

現時点では `aws_vpc` 1個に対して複数の `aws_subnet`, `aws_security_group`, `aws_vpc_endpoint` (Interface 型, Gateway 型), `aws_route_table` を作成することができます。

また、インデックスにナンバリングを使わないことに関しては、ナンバリングでなければいいので文字列にすることで解決しました。これにより Terraform 上のリソース名も `module.vpc.aws_subnet.subnets["public-secondary"]` となるため、削除する際にはリソースの前後関係を意識せずに削除することができます。

# おわりに

こんな感じで、最近感じた違和感といい感じの解決策を書いてみました。

あまり Terraform Module を使わずに直接リソース書く方が楽なんですが（諸説あり）、どうせ書くなら変更しやすくて融通の利く Terraform Module を作りたいものです。
