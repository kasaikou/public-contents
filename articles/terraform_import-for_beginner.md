---
title: 未知のサービスに対するTerraform初心者アプローチ
emoji: 🛵
type: tech
topics: [terraform, AWS]
published: false
---

# 私を含めた最初からTerraformを使おうとして挫折した方々，いかがお過ごしでしょうか
この記事では，Terraformで未知のリソースを定義しようと思ったときに私がしていることを紹介しています．読者対象としては「Terraformを導入したはいいけれど，リソースの定義方法がわからないまま挫折して自然消滅させた方」を対象としています．まあ，Terraformの公式ドキュメントを読んでわかる人は直接の対象ではありませんが，「こういう人もいるんだな」くらいで見ていただければ幸いです．

口を大にして言いますが，この記事の合言葉は **「公式ドキュメントを読んでわかるならそれに越したことはない」** です．そのうえで，公式ドキュメントを読んでわからなかった人が，Terraformを利用するるための足がかりをこの記事ではECS on Fargateのサービス作成とS3バケットの作成を例に挙げながら提案していきます．

Repeat after me, **「公式ドキュメントを読んでわかるならそれに越したことはない」**

# TL;DR
- 最初からTerraformを使うだなんて，そんな無理をする必要はないんだよ
    - 一度マネジメントコンソールでリソースを作ったらいいんだよ
- `terraform import` を用いることで，すでにあるリソースを取り込むことができるよ
- `terraform plan` を用いることで，リソースの状態と `.tf` ファイルで定義している内容を比較することができるよ
    - 差がなくなるように調整することでリソースを再現することができるね
- この方法によって作成されるリソースは完全に再現しているとは言えないよ
    - Terraform で定義したファイルからリソースを作成して，ちゃんとチェックする必要があるね

# 最初からTerraformを使うだなんて，そんな無理をする必要はない
TL;DRでも基本方針は示しましたが，改めて，今回の方法について全体像を確認します．

---
基本的に， **公式ドキュメントを読んでわかるならそれに越したことはない** です．Terraform の公式ドキュメントはサンプル付きで比較的読みやすいドキュメントであるため，それをそのまま利用したとしても，かなり期待に添えたものをつくることができるでしょう．しかし，そのインフラリソースに対する知識が必要なものや，複数のリソースを作成しなければいけないものなど，一筋縄ではいかないものもかなりあります．これらを何も知らない状態からTerraformだけで定義するのは至難の業です．そこで，最初からTerraformを使わずに一度マネジメントコンソールでリソースを作成します．その後， `terraform import` でリソースをTerraformプロジェクトに取り込み，`terraform plan` で差分を比較しながらプロジェクトを編集していくのです．

一度，マネジメントコンソールでプロジェクトにリソースを定義することで複雑なリソースをテンプレート的に起動させ， `terraform import` を用いてTerraformプロジェクトにリソースを取り込むことで，マネジメントコンソールで作製した場合のリソースをTerraformに取り込むことができます．これを `terraform plan` することで，定義されたリソースと現在のリソースとの差分を得られるので，この差分がなくなるように修正していきます．

## 一度マネジメントコンソールでリソースを作る

はじめに，ECSクラスターを作成します．とりあえず作ってみましょう．

![](https://storage.googleapis.com/zenn-user-upload/8a536371b440-20221215.png)

![](https://storage.googleapis.com/zenn-user-upload/aa97c1fd69cd-20221215.png)

なんとなくメトリクスを有効にしてみました．

![](https://storage.googleapis.com/zenn-user-upload/588a84728f0c-20221215.png)

これでECSクラスターが完成しました．次に，タスク定義を見ていきます．タスク定義は文字通り，ECSクラスタ内で実行するタスクの定義なのでタスク定義です．ここで実行するコンテナについて定義していきます．今回は `httpd:2.4` を使います．

![](https://storage.googleapis.com/zenn-user-upload/3fe3797b7643-20221215.png)

![](https://storage.googleapis.com/zenn-user-upload/59b79d97cbd0-20221215.png)

最後に，サービスを作成します．タスク定義を使ってECSクラスターに実際に稼働するコンテナを作ります．

![](https://storage.googleapis.com/zenn-user-upload/84f874b40bd7-20221215.png)

![](https://storage.googleapis.com/zenn-user-upload/eb9ca5b6c83e-20221215.png)

![](https://storage.googleapis.com/zenn-user-upload/8218cf02a0a0-20221215.png)

あとで直接実行を確認するためにパブリックIPを割り当てました．

これでサービスが完成しました．

![](https://storage.googleapis.com/zenn-user-upload/5ebeaf7e68f1-20221215.png)

サービスの実体は永続稼働するタスクのようで，タスクに移動すると実行中のタスクについて確認できます．

![](https://storage.googleapis.com/zenn-user-upload/0dab1b58f34e-20221215.png)

もちろん，パブリックIPにアクセスするとちゃんとApacheが稼働していることを確認することができます．

![](https://storage.googleapis.com/zenn-user-upload/482b67ad0c60-20221215.png)

## `terraform import` & `terraform plan`

次に，先ほど作ったリソースをTerraformにインポートしましょう．既存のプロジェクトがある場合には影響を受けないように独立したプロジェクトとして作ることをお勧めします．
インポート先となるTerraformリソースを作らないとインポートできないので，形だけ作ります．今回作成したリソースはECSクラスター，タスク定義，サービスの3種類でした．それっぽそうなリソースをドキュメントから探してそれっぽく定義します．

![](https://storage.googleapis.com/zenn-user-upload/a64d7e3f1a6b-20221215.png)

```tf
terraform {}

provider "aws" {
  region = "ap-northeast-1"
}

resource "aws_ecs_cluster" "test" {}

resource "aws_ecs_task_definition" "test" {}

resource "aws_ecs_service" "test" {}
```

次に， `terraform import` で既存リソースを取り込みます．
```sh
$ terraform import <Terraformにおけるリソース名> <ARNなどのリソースを示す固有表現>
```

です．サクっとimportしていきます．

```sh
$ terraform import aws_ecs_cluster.test example-advent-calendar
$ terraform import aws_ecs_task_definition.test arn:aws:ecs:ap-northeast-1:${ACCOUNT_ID}:task-definition/example-advent-calendar:1
$ terraform import aws_ecs_service.test example-advent-calendar/tes
```

続いて，`terraform plan` で変更差分を観察します．

```log
$ terraform plan
╷
│ Error: Missing required argument
│ 
│   on main.tf line 7, in resource "aws_ecs_cluster" "test":
│    7: resource "aws_ecs_cluster" "test" {}
│ 
│ The argument "name" is required, but no definition was found.
╵
╷
│ Error: Missing required argument
│ 
│   on main.tf line 9, in resource "aws_ecs_task_definition" "test":
│    9: resource "aws_ecs_task_definition" "test" {}
│ 
│ The argument "container_definitions" is required, but no definition was found.
╵
╷
│ Error: Missing required argument
│ 
│   on main.tf line 9, in resource "aws_ecs_task_definition" "test":
│    9: resource "aws_ecs_task_definition" "test" {}
│ 
│ The argument "family" is required, but no definition was found.
╵
╷
│ Error: Missing required argument
│ 
│   on main.tf line 11, in resource "aws_ecs_service" "test":
│   11: resource "aws_ecs_service" "test" {}
│ 
│ The argument "name" is required, but no definition was found.
╵
```

エラーが出てしまいました．必須の引数が設定されていないということなので，エラーが出ないように少し修正していきます．

```tf
resource "aws_ecs_cluster" "test" {
  name = "example-advent-calendar"
}

resource "aws_ecs_task_definition" "test" {
  family                = "example-advent-calendar"
  container_definitions = jsonencode({})
}

resource "aws_ecs_service" "test" {
  name = "test"
}
```

もう一度コマンドを実行していきます．
```log
$ terraform plan
Terraform used the selected providers to generate the following execution plan. Resource actions are
indicated with the following symbols:
  ~ update in-place
-/+ destroy and then create replacement

Terraform will perform the following actions:

  # aws_ecs_cluster.test will be updated in-place
  ~ resource "aws_ecs_cluster" "test" {
        id                 = "arn:aws:ecs:ap-northeast-1:${ACCOUNT_ID}:cluster/example-advent-calendar"
        name               = "example-advent-calendar"
      ~ tags               = {
          - "ecs:cluster:createdFrom" = "ecs-console-v2" -> null
        }
      ~ tags_all           = {
          - "ecs:cluster:createdFrom" = "ecs-console-v2"
        } -> (known after apply)
        # (2 unchanged attributes hidden)

      - configuration {
          - execute_command_configuration {
              - logging = "DEFAULT" -> null
            }
        }

      - service_connect_defaults {
          - namespace = "arn:aws:servicediscovery:ap-northeast-1:${ACCOUNT_ID}:namespace/ns-redlrj27ommn6ful" -> null
        }

        # (1 unchanged block hidden)
    }

  # aws_ecs_service.test must be replaced
-/+ resource "aws_ecs_service" "test" {
      ~ cluster                            = "arn:aws:ecs:ap-northeast-1:${ACCOUNT_ID}:cluster/example-advent-calendar" -> (known after apply)
      ~ deployment_maximum_percent         = 100 -> 200
      ~ deployment_minimum_healthy_percent = 0 -> 100
      - desired_count                      = 1 -> null
      ~ enable_ecs_managed_tags            = true -> false
      - health_check_grace_period_seconds  = 0 -> null
      ~ iam_role                           = "aws-service-role" -> (known after apply)
      ~ id                                 = "arn:aws:ecs:ap-northeast-1:${ACCOUNT_ID}:service/example-advent-calendar/test" -> (known after apply)
      + launch_type                        = (known after apply)
        name                               = "test"
      ~ platform_version                   = "LATEST" -> (known after apply)
      - propagate_tags                     = "NONE" -> null
      - tags                               = {
          - "ecs:service:stackId" = "arn:aws:cloudformation:ap-northeast-1:${ACCOUNT_ID}:stack/ECS-Console-V2-Service-683d2ac3-9d9e-4268-8f55-da09a31f83b9/0862c090-7c79-11ed-84e2-065ee58bc9a3"
        } -> null
      ~ tags_all                           = {
          - "ecs:service:stackId" = "arn:aws:cloudformation:ap-northeast-1:${ACCOUNT_ID}:stack/ECS-Console-V2-Service-683d2ac3-9d9e-4268-8f55-da09a31f83b9/0862c090-7c79-11ed-84e2-065ee58bc9a3"
        } -> (known after apply)
      - task_definition                    = "example-advent-calendar:1" -> null
      ~ triggers                           = {} -> (known after apply)
      + wait_for_steady_state              = false
        # (2 unchanged attributes hidden)

      - capacity_provider_strategy { # forces replacement
          - base              = 0 -> null
          - capacity_provider = "FARGATE" -> null
          - weight            = 1 -> null
        }

      - deployment_circuit_breaker {
          - enable   = true -> null
          - rollback = true -> null
        }

      - deployment_controller {
          - type = "ECS" -> null
        }

      - network_configuration {
          - assign_public_ip = true -> null
          - security_groups  = [
              - "sg-XXXXXXXX",
            ] -> null
          - subnets          = [
              - "subnet-XXXXXXXX",
              - "subnet-XXXXXXXX",
              - "subnet-XXXXXXXX",
            ] -> null
        }

      - timeouts {}
    }

  # aws_ecs_task_definition.test must be replaced
-/+ resource "aws_ecs_task_definition" "test" {
      ~ arn                      = "arn:aws:ecs:ap-northeast-1:${ACCOUNT_ID}:task-definition/example-advent-calendar:1" -> (known after apply)
      ~ container_definitions    = jsonencode(
          ~ [ # forces replacement
              - {
                  - command               = []
                  - cpu                   = 0
                  - dnsSearchDomains      = []
                  - dnsServers            = []
                  - dockerLabels          = {}
                  - dockerSecurityOptions = []
                  - entryPoint            = []
                  - environment           = []
                  - environmentFiles      = []
                  - essential             = true
                  - extraHosts            = []
                  - image                 = "httpd:2.4"
                  - links                 = []
                  - logConfiguration      = {
                      - logDriver     = "awslogs"
                      - options       = {
                          - awslogs-create-group  = "true"
                          - awslogs-group         = "/ecs/example-advent-calendar"
                          - awslogs-region        = "ap-northeast-1"
                          - awslogs-stream-prefix = "ecs"
                        }
                      - secretOptions = []
                    }
                  - mountPoints           = []
                  - name                  = "apache"
                  - portMappings          = [
                      - {
                          - appProtocol   = "http"
                          - containerPort = 80
                          - hostPort      = 80
                          - name          = "apache-80-tcp"
                          - protocol      = "tcp"
                        },
                    ]
                  - secrets               = []
                  - systemControls        = []
                  - ulimits               = []
                  - volumesFrom           = []
                },
            ]
        )
      - cpu                      = "256" -> null # forces replacement
      - execution_role_arn       = "arn:aws:iam::${ACCOUNT_ID}:role/ecsTaskExecutionRole" -> null # forces replacement
      ~ id                       = "example-advent-calendar" -> (known after apply)
      - memory                   = "512" -> null # forces replacement
      ~ network_mode             = "awsvpc" -> (known after apply)
      - requires_compatibilities = [
          - "FARGATE",
        ] -> null # forces replacement
      ~ revision                 = 1 -> (known after apply)
      + skip_destroy             = false
      - tags                     = {
          - "ecs:taskDefinition:createdFrom" = "ecs-console-v2"
          - "ecs:taskDefinition:stackId"     = "arn:aws:cloudformation:ap-northeast-1:${ACCOUNT_ID}:stack/ECS-Console-V2-TaskDefinition-128d7169-5779-4f50-ba1c-362d3f785f16/53879e70-7c78-11ed-a8b3-0a4c0c106c7d"
        } -> null
      ~ tags_all                 = {
          - "ecs:taskDefinition:createdFrom" = "ecs-console-v2"
          - "ecs:taskDefinition:stackId"     = "arn:aws:cloudformation:ap-northeast-1:${ACCOUNT_ID}:stack/ECS-Console-V2-TaskDefinition-128d7169-5779-4f50-ba1c-362d3f785f16/53879e70-7c78-11ed-a8b3-0a4c0c106c7d"
        } -> (known after apply)
        # (1 unchanged attribute hidden)

      - runtime_platform { # forces replacement
          - cpu_architecture        = "X86_64" -> null # forces replacement
          - operating_system_family = "LINUX" -> null # forces replacement
        }
    }

Plan: 2 to add, 1 to change, 2 to destroy.
```

色々な変更が出力されました．これとマネージメントコンソールの情報を基に変更差分がなくなるようなリソースを書いていきます．といっても， `terraform plan` というコマンド名なだけあって，差分で出てくる引数名や表現はそのままTerraformでも使えます．

この時に，マネージメントコンソールでの表現とTerraformのリソースドキュメントと照らし合わせることで，Terraformドキュメントを効率よく読み進められます．この記事ではTerraformリソースで再現することを焦点に当てていますが．次の段階としてTerraformからリソースを修正していく際に直感的に変更を加えやすくなるのです．

:::message
今回は直接影響しないのでタグは削除する方向で進めていきます．
:::

```tf
resource "aws_ecs_cluster" "test" {
  name = "example-advent-calendar"
  configuration {
    execute_command_configuration {
      logging = "DEFAULT"
    }
  }
  service_connect_defaults {
    namespace = "arn:aws:servicediscovery:ap-northeast-1:${ACCOUNT_ID}:namespace/ns-redlrj27ommn6ful"
  }
}

resource "aws_ecs_task_definition" "test" {
  family                   = "example-advent-calendar"
  cpu                      = "256"
  memory                   = "512"
  network_mode             = "awsvpc"
  requires_compatibilities = ["FARGATE"]
  execution_role_arn       = "arn:aws:iam::${ACCOUNT_ID}:role/ecsTaskExecutionRole"
  task_role_arn            = "arn:aws:iam::${ACCOUNT_ID}:role/ecsTaskExecutionRole"
  runtime_platform {
    cpu_architecture        = "X86_64"
    operating_system_family = "LINUX"
  }
  container_definitions = jsonencode([
    {
      essential = true
      name      = "apache"
      image     = "httpd:2.4"
      logConfiguration = {
        logDriver = "awslogs"
        options = {
          awslogs-create-group  = "true"
          awslogs-group         = "/ecs/example-advent-calendar"
          awslogs-region        = "ap-northeast-1"
          awslogs-stream-prefix = "ecs"
        }
      }
      portMappings = [{
        appProtocol   = "http"
        containerPort = 80
        hostPort      = 80
        name          = "apache-80-tcp"
        protocol      = "tcp"
      }]
    }
  ])
}

resource "aws_ecs_service" "test" {
  name                               = "test"
  cluster                            = aws_ecs_cluster.test.name
  task_definition                    = aws_ecs_task_definition.test.arn
  deployment_maximum_percent         = 100
  deployment_minimum_healthy_percent = 0
  desired_count                      = 1
  enable_ecs_managed_tags            = true
  health_check_grace_period_seconds  = 0
  propagate_tags                     = "NONE"
  deployment_circuit_breaker {
    enable   = true
    rollback = true
  }
  deployment_controller {
    type = "ECS"
  }
  capacity_provider_strategy {
    capacity_provider = "FARGATE"
    base              = 0
    weight            = 1
  }
  network_configuration {
    assign_public_ip = true
    security_groups = [
      "sg-XXXXXXXX",
    ]
    subnets = [
      "subnet-XXXXXXXX",
      "subnet-XXXXXXXX",
      "subnet-XXXXXXXX",
    ]
  }
}
```

もう一度 `terraform plan` で変更差分を観察します．

```log
$ terraform plan
Terraform used the selected providers to generate the following execution plan. Resource actions are
indicated with the following symbols:
  ~ update in-place
-/+ destroy and then create replacement

Terraform will perform the following actions:

  # aws_ecs_cluster.test will be updated in-place
  ~ resource "aws_ecs_cluster" "test" {
        id                 = "arn:aws:ecs:ap-northeast-1:${ACCOUNT_ID}:cluster/example-advent-calendar"
        name               = "example-advent-calendar"
      ~ tags               = {
          - "ecs:cluster:createdFrom" = "ecs-console-v2" -> null
        }
      ~ tags_all           = {
          - "ecs:cluster:createdFrom" = "ecs-console-v2"
        } -> (known after apply)
        # (2 unchanged attributes hidden)

        # (3 unchanged blocks hidden)
    }

  # aws_ecs_service.test must be replaced
-/+ resource "aws_ecs_service" "test" {
      ~ cluster                            = "arn:aws:ecs:ap-northeast-1:${ACCOUNT_ID}:cluster/example-advent-calendar" -> "example-advent-calendar" # forces replacement
      ~ id                                 = "arn:aws:ecs:ap-northeast-1:${ACCOUNT_ID}:service/example-advent-calendar/test" -> (known after apply)
      + launch_type                        = (known after apply)
        name                               = "test"
      ~ platform_version                   = "LATEST" -> (known after apply)
      - tags                               = {
          - "ecs:service:stackId" = "arn:aws:cloudformation:ap-northeast-1:${ACCOUNT_ID}:stack/ECS-Console-V2-Service-683d2ac3-9d9e-4268-8f55-da09a31f83b9/0862c090-7c79-11ed-84e2-065ee58bc9a3"
        } -> null
      ~ tags_all                           = {
          - "ecs:service:stackId" = "arn:aws:cloudformation:ap-northeast-1:${ACCOUNT_ID}:stack/ECS-Console-V2-Service-683d2ac3-9d9e-4268-8f55-da09a31f83b9/0862c090-7c79-11ed-84e2-065ee58bc9a3"
        } -> (known after apply)
      ~ task_definition                    = "example-advent-calendar:1" -> (known after apply)
      ~ triggers                           = {} -> (known after apply)
      + wait_for_steady_state              = false
        # (8 unchanged attributes hidden)

      - timeouts {}

        # (4 unchanged blocks hidden)
    }

  # aws_ecs_task_definition.test must be replaced
-/+ resource "aws_ecs_task_definition" "test" {
      ~ arn                      = "arn:aws:ecs:ap-northeast-1:${ACCOUNT_ID}:task-definition/example-advent-calendar:1" -> (known after apply)
      ~ container_definitions    = jsonencode(
          ~ [ # forces replacement
              ~ {
                  - command               = [] -> null
                  - cpu                   = 0 -> null
                  - dnsSearchDomains      = [] -> null
                  - dnsServers            = [] -> null
                  - dockerLabels          = {} -> null
                  - dockerSecurityOptions = [] -> null
                  - entryPoint            = [] -> null
                  - environment           = [] -> null
                  - environmentFiles      = [] -> null
                  - extraHosts            = [] -> null
                  - links                 = [] -> null
                  ~ logConfiguration      = {
                      - secretOptions = [] -> null
                        # (2 unchanged elements hidden)
                    }
                  - mountPoints           = [] -> null
                    name                  = "apache"
                  - secrets               = [] -> null
                  - systemControls        = [] -> null
                  - ulimits               = [] -> null
                  - volumesFrom           = [] -> null
                    # (3 unchanged elements hidden)
                } # forces replacement,
            ]
        )
      ~ id                       = "example-advent-calendar" -> (known after apply)
      ~ revision                 = 1 -> (known after apply)
      + skip_destroy             = false
      - tags                     = {
          - "ecs:taskDefinition:createdFrom" = "ecs-console-v2"
          - "ecs:taskDefinition:stackId"     = "arn:aws:cloudformation:ap-northeast-1:${ACCOUNT_ID}:stack/ECS-Console-V2-TaskDefinition-128d7169-5779-4f50-ba1c-362d3f785f16/53879e70-7c78-11ed-a8b3-0a4c0c106c7d"
        } -> null
      ~ tags_all                 = {
          - "ecs:taskDefinition:createdFrom" = "ecs-console-v2"
          - "ecs:taskDefinition:stackId"     = "arn:aws:cloudformation:ap-northeast-1:${ACCOUNT_ID}:stack/ECS-Console-V2-TaskDefinition-128d7169-5779-4f50-ba1c-362d3f785f16/53879e70-7c78-11ed-a8b3-0a4c0c106c7d"
        } -> (known after apply)
        # (6 unchanged attributes hidden)

        # (1 unchanged block hidden)
    }

Plan: 2 to add, 1 to change, 2 to destroy.
```

とりあえず，変更差分が減りましたが，Terraformのファイル内にARNが表にあるのはどうにも気に入りません．どうやら，他に依存しているリソースがある様です．この依存しているリソースもAWSが提供しているものでない限りは積極的に取り込んでしまいましょう．また，AWSが提供しているものであっても `Data Source` として取り込むことができます．これを繰り返してできた Terraform のコードが以下になります．IAMロールなどのARNが隠されていい感じになってきました．

ただ，ECS名前空間は機能が最新すぎてTerraformリソースとして作ることができません．正直，しばらく名前空間使う予定ないので消すことにしました（再現するのに不用意に消していくスタイル）．

あまりにも長いので今回は差分だけ表示します．

```diff tf
6a7,34
> data "aws_vpc" "default_apne1" {
>   default = true
> }
> 
> data "aws_security_group" "default_apne1" {
>   vpc_id = data.aws_vpc.default_apne1.id
>   name   = "default"
> }
> 
> data "aws_subnet" "default_apne1a" {
>   availability_zone = "ap-northeast-1a"
>   vpc_id            = data.aws_vpc.default_apne1.id
> }
> 
> data "aws_subnet" "default_apne1c" {
>   availability_zone = "ap-northeast-1c"
>   vpc_id            = data.aws_vpc.default_apne1.id
> }
> 
> data "aws_subnet" "default_apne1d" {
>   availability_zone = "ap-northeast-1d"
>   vpc_id            = data.aws_vpc.default_apne1.id
> }
> 
> data "aws_iam_role" "ecs_exec" {
>   name = "ecsTaskExecutionRole"
> }
> 
14,16d41
<   service_connect_defaults {
<     namespace = "arn:aws:servicediscovery:ap-northeast-1:${ACCOUNT_ID}:namespace/ns-redlrj27ommn6ful"
<   }
25,26c50,51
<   execution_role_arn       = "arn:aws:iam::${ACCOUNT_ID}:role/ecsTaskExecutionRole"
<   task_role_arn            = "arn:aws:iam::${ACCOUNT_ID}:role/ecsTaskExecutionRole"
---
>   execution_role_arn       = data.aws_iam_role.ecs_exec.arn
>   task_role_arn            = data.aws_iam_role.ecs_exec.arn
81c106
<       "sg-XXXXXXXX",
---
>       data.aws_security_group.default_apne1.id,
84,86c109,111
<       "subnet-XXXXXXXX",
<       "subnet-XXXXXXXX",
<       "subnet-XXXXXXXX",
---
>       data.aws_subnet.default_apne1a.id,
>       data.aws_subnet.default_apne1c.id,
>       data.aws_subnet.default_apne1d.id,
```

もう一度 `terraform plan` で変更差分を観察します．
```log
$ terraform plan
Terraform used the selected providers to generate the
following execution plan. Resource actions are indicated
with the following symbols:
  ~ update in-place
-/+ destroy and then create replacement

Terraform will perform the following actions:

  # aws_ecs_cluster.test will be updated in-place
  ~ resource "aws_ecs_cluster" "test" {
        id                 = "arn:aws:ecs:ap-northeast-1:${ACCOUNT_ID}:cluster/example-advent-calendar"
        name               = "example-advent-calendar"
      ~ tags               = {
          - "ecs:cluster:createdFrom" = "ecs-console-v2" -> null
        }
      ~ tags_all           = {
          - "ecs:cluster:createdFrom" = "ecs-console-v2"
        } -> (known after apply)
        # (2 unchanged attributes hidden)

      - service_connect_defaults {
          - namespace = "arn:aws:servicediscovery:ap-northeast-1:${ACCOUNT_ID}:namespace/ns-redlrj27ommn6ful" -> null
        }

        # (2 unchanged blocks hidden)
    }

  # aws_ecs_service.test must be replaced
-/+ resource "aws_ecs_service" "test" {
      ~ cluster                            = "arn:aws:ecs:ap-northeast-1:${ACCOUNT_ID}:cluster/example-advent-calendar" -> "example-advent-calendar" # forces replacement
      ~ id                                 = "arn:aws:ecs:ap-northeast-1:${ACCOUNT_ID}:service/example-advent-calendar/test" -> (known after apply)
      + launch_type                        = (known after apply)
        name                               = "test"
      ~ platform_version                   = "LATEST" -> (known after apply)
      - tags                               = {
          - "ecs:service:stackId" = "arn:aws:cloudformation:ap-northeast-1:${ACCOUNT_ID}:stack/ECS-Console-V2-Service-683d2ac3-9d9e-4268-8f55-da09a31f83b9/0862c090-7c79-11ed-84e2-065ee58bc9a3"
        } -> null
      ~ tags_all                           = {
          - "ecs:service:stackId" = "arn:aws:cloudformation:ap-northeast-1:${ACCOUNT_ID}:stack/ECS-Console-V2-Service-683d2ac3-9d9e-4268-8f55-da09a31f83b9/0862c090-7c79-11ed-84e2-065ee58bc9a3"
        } -> (known after apply)
      ~ task_definition                    = "example-advent-calendar:1" -> (known after apply)
      ~ triggers                           = {} -> (known after apply)
      + wait_for_steady_state              = false
        # (8 unchanged attributes hidden)

      - timeouts {}

        # (4 unchanged blocks hidden)
    }

  # aws_ecs_task_definition.test must be replaced
-/+ resource "aws_ecs_task_definition" "test" {
      ~ arn                      = "arn:aws:ecs:ap-northeast-1:${ACCOUNT_ID}:task-definition/example-advent-calendar:1" -> (known after apply)
      ~ container_definitions    = jsonencode(
          ~ [ # forces replacement
              ~ {
                  - command               = [] -> null
                  - cpu                   = 0 -> null
                  - dnsSearchDomains      = [] -> null
                  - dnsServers            = [] -> null
                  - dockerLabels          = {} -> null
                  - dockerSecurityOptions = [] -> null
                  - entryPoint            = [] -> null
                  - environment           = [] -> null
                  - environmentFiles      = [] -> null
                  - extraHosts            = [] -> null
                  - links                 = [] -> null
                  ~ logConfiguration      = {
                      - secretOptions = [] -> null
                        # (2 unchanged elements hidden)
                    }
                  - mountPoints           = [] -> null
                    name                  = "apache"
                  - secrets               = [] -> null
                  - systemControls        = [] -> null
                  - ulimits               = [] -> null
                  - volumesFrom           = [] -> null
                    # (3 unchanged elements hidden)
                } # forces replacement,
            ]
        )
      ~ id                       = "example-advent-calendar" -> (known after apply)
      ~ revision                 = 1 -> (known after apply)
      + skip_destroy             = false
      - tags                     = {
          - "ecs:taskDefinition:createdFrom" = "ecs-console-v2"
          - "ecs:taskDefinition:stackId"     = "arn:aws:cloudformation:ap-northeast-1:${ACCOUNT_ID}:stack/ECS-Console-V2-TaskDefinition-128d7169-5779-4f50-ba1c-362d3f785f16/53879e70-7c78-11ed-a8b3-0a4c0c106c7d"
        } -> null
      ~ tags_all                 = {
          - "ecs:taskDefinition:createdFrom" = "ecs-console-v2"
          - "ecs:taskDefinition:stackId"     = "arn:aws:cloudformation:ap-northeast-1:${ACCOUNT_ID}:stack/ECS-Console-V2-TaskDefinition-128d7169-5779-4f50-ba1c-362d3f785f16/53879e70-7c78-11ed-a8b3-0a4c0c106c7d"
        } -> (known after apply)
        # (6 unchanged attributes hidden)

        # (1 unchanged block hidden)
    }

Plan: 2 to add, 1 to change, 2 to destroy.
```

変更差分は同じですが，依存する外部のリソースもおそらく大半のものは回収でき，回収できないものも `Data Source` として取り込むことができました．つまり，先ほどマネージメントコンソールで作成したリソースがTerraformのリソースとしてだいたいの再現を行うことができたのです．

:::message alert
もちろん，何度か方針の変更が行われたので厳密には違うものです．ちゃんと `terraform apply` をしてTerraformで作成したリソースがどのような状態になっているのかを確認した方がよいと思います．
:::

完成したら，もうリソースは使用しないので削除します． `terraform destroy` でリソースを一括削除できるのでサクっとしてしまいましょう．その際に余計なリソースを削除してはいけないので，削除するリソースが何であるかを確認しておいてください．

```
$ terraform destroy
```

## 閑話: AWSのタグには何を入れるか

Terraformでリソースを管理する目的は色々あると思いますが，その中の一つに「リソースタグをつけたり，機械的な作業を自動化したい」という目的があると思われます．その際にどのようなタグを付けるべきか，という議論をちょくちょく見かけるので，私の一意見として書いておきます．これは私の個人的に使用しているAWSアカウントに対して適用しているものです．

| タグ名 | どのようなものか | 値の一例 |
| :-: | :-- | :-- |
| `ProjectName` | どのような目的で作成したものかを記録 | 任意のプロジェクト名（プロダクト名）
| `ManagedBy` | どのような管理方法を行っているかを記録（設定されていない場合は手作業で作成したものとみなす） | `Terraform`, `CloudFormation` など
| `RepositoryURL` | これを管理しているリポジトリURLを記録 | `github.com/example/repository` など

TerraformでAWSタグを設定する方法には，プロバイダに `shared_tags` を設定する方法と，リソースごとに `tags` を設定する方法の2種類が考えられます．今現在，私は辞書型の変数をTerraformに設置して各リソースごとにAWSタグを設定する方法をとっています．

```tf
locals {
  tags = {
    ProjectName = "AdventCalendar2022"
    ManagedBy = "Terraform"
  }
}

resource "aws_foo_bar" "piyo" {
  tags = merge(local.tags, {
    Name = "piyo"
  })
}
```

この辺は，それぞれいろいろな考え方があるので私一個人で「これが絶対いいです」とは言えないんですが，以下のように比較的簡単にmergeすることができ，モジュールごとに分割したあたりから，修正を加えやすく管理しやすい状態にできるのが今のスタイルのメリットだと思っています．

# この方法は完ぺきではない

さて，ここまで比較的簡単な方法でTerraformプロジェクトに落とし込むことを行いましたが，次にこの方法の良くない点についてです．以下のS3バケットの作成の例で見ていきます．先ほどと同じように，マネージメントコンソールでバケットを作り，また，Terraformのリソースとして設定していきます．

![](https://storage.googleapis.com/zenn-user-upload/7dface415cbf-20221216.png)

```tf
resource "aws_s3_bucket" "bucket" {
  bucket = "terraform-advent-calendar-2022"
}
```

importとplanをしてみます．

```log
$ terraform import aws_s3_bucket.bucket terraform-advent-calendar-2022
$ terraform plan
No changes. Your infrastructure matches the configuration.
```

どうやら完成したようです．

## 検証を行う

さて，これをベースにリソースを作ってみます．バケット名は全世界で唯一ですのでバケット名だけ変えてリソースを作成します．

```tf
resource "aws_s3_bucket" "bucket" {
  bucket = "terraform-advent-calendar-2022"
}

resource "aws_s3_bucket" "bucket_copy" { # 追加
  bucket = "terraform-advent-calendar-2022-copy"
}

```

```log
$ terraform apply
```

マネージメントコンソールを見て見ます．

![](https://storage.googleapis.com/zenn-user-upload/5386c55e18df-20221216.png)

とりあえず問題なくリソースを作成できていそう．しかし，次です．

![](https://storage.googleapis.com/zenn-user-upload/a9786b214311-20221216.png)

このように，若干の際が発生してしまうのです．今回の場合，バケットへのアクセス許可に関する設定のところで差分が出てしまいました．このように，Terraformから作るリソースとマネージメントコンソール空作成するリソースで差異が生じることがあります．そのため， `terraform import` したリソースは同様のものをTerraform からさくせいすることができるか，という検証が必要なのです．

:::message
ちなみに，前章のECSも `terraform plan` で示された差分にあるものを追加するだけだと失敗します．最初の `terraform plan` に載っていないが，最終的に追加されているオプションがあります．何度か調整したので `terraform plan` で矛盾が生じているかもしれませんが許してください．

なんとなく確認してみたいという人のために，答えは隠しておきます．
:::details 答え
（少なくとも `hashicorp/aws v4.47.0` において，）タスク定義 `aws_ecs_task_definition` にあるタスクロール `task_role_arn` が表示されません．確認してみてください．
:::
:::

それでは修正していきます．

```tf
resource "aws_s3_bucket" "bucket" {
  bucket = "terraform-advent-calendar-2022"
}

resource "aws_s3_bucket" "bucket_copy" { # 
  bucket = "terraform-advent-calendar-2022-copy"
}

resource "aws_s3_bucket_public_access_block" "bucket_copy" {
  bucket = aws_s3_bucket.bucket_copy.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

新たに， `aws_s3_bucket_public_access_block` という名前のリソースを作成しました．これでもう一度Terraformから更新を行い，マネージメントコンソールで確認します．

```
$ terraform apply
```

![](https://storage.googleapis.com/zenn-user-upload/97996717f8cf-20221216.png)

これでおそらく同一のものができたと思われます（と言いながら自信がないのはECSのところを含め，かなり急ピッチで作ったので他にも見落としがないか不安だからです．何か見つけたら教えてください）．

最後はECSの時と同様に， `terraform destroy` でリソースを削除して終わりにします．

# まとめ？
このように， `terraform import` でリソースを追加することができます．これは直感的な理解にとても優位に働き，少なくとも Terraform を初めて使う人にとっては直感的にリソースを理解する足がかりとなります．しかし，それに対する検証が必ずと言っていいほど必要になるので忘れずに行ってください．

長くなりましたが，最後までありがとうございました．
