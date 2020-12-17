# **ECS-CodeDeploy**

<br>

## **About**

先程作成したCodebuildにECSでのDeployフェーズを追加する

<br>

## **Inside Package**
 * web nginx:alpine
 * app php
 * DBはAWSのRDSを使用

<br>

## **手順**

<br>

### 0. RDS構築済みであること

Cloudformation で RDSを作成

<br>

### 1. ECS Clusterの作成

* Amazon ECS > クラスター > クラスターの作成
<br>クラスターテンプレート
<br>EC2 Linux + ネットワーキング
<br>
<br>クラスターの設定
<br>クラスター名　ecs-cluster（例）
<br>
<br>インスタンスの設定
<br>EC2 インスタンスタイプ　t2.micro
<br>インスタンス数　2 ※t2.micro 1台ではDeployのメモリ不足
<br>キーペア　自身のSSH-KEY
<br>
<br>ネットワーキング
<br>VPC　既存VPCを選択、RDSと同一VPCであること
<br>サブネット　パブリック
<br>セキュリティグループ　WEB・SSHの許可
<br>

* 確認
<br>クラスター > ecs-cluster　のステータス画面で
<br>EC2 > コンテナインスタンス が 2 であること

### 2. タスク定義の作成

* Amazon ECS > タスク定義 > 新しいタスク定義の作成
<br>
<br>起動タイプ　EC2
<br>
<br>タスクとコンテナの定義の設定
<br>タスク定義名　ecs-cluster-task
<br>タスクロール　ecsTaskExecutionRole
<br>
<br>コンテナの定義 > コンテナの追加
<br>
<br>webコンテナの設定
<br>スタンダード
<br>コンテナ名　web
<br>イメージ  ECRレポジトリのwebイメージのURIをコピペ
<br>メモリ制限 (MiB)*　ハード制限・300
<br>ポートマッピング　80:80
<br>
<br>ネットワーク設定
<br>リンク　app:app
<br>
<br>appコンテナの設定
<br>スタンダード
<br>コンテナ名　app
<br>イメージ  ECRレポジトリのappイメージのURIをコピペ
<br>メモリ制限 (MiB)*　ハード制限・600
<br>
<br>環境
<br>環境変数でRDS接続パラメータを設定
<br>DB_HOST　Value　RDSのエンドポイントをコピペ
<br>DB_NAME　Value　RDSで作成したデータベース名
<br>DB_USER　Value　RDSで作成したユーザー名
<br>DB_PASSWORD　Value　RDSで作成したパスワード
<br>
<br>作成

* 続けてサービスの作成　アクション > サービスの作成
<br>
<br>起動タイプ　EC2
<br>サービス名　laravel-app-ecs（例）
<br>サービスタイプ　REPLICA
<br>タスクの数　1
<br>デプロイメントタイプ　ローリングアップデート
<br>
<br>次のステップで画面を進め、サービスの作成
<br>

* 確認
<br>クラスター > ecs-cluster　のステータス画面で
<br>EC2 > 実行中のタスク が 1 であること

### 3. CodeDeployの作成

* デベロッパー用ツール > CodePipeline > パイプライン
<br>
<br>前回作成したCodebuildのパイプラインを選択
<br>
<br>編集する
<br>Buildの後で　＋ステージを追加する　を選択
<br>ステージ名　Deploy
<br>＋アクショングループを追加する　を選択
<br>アクション名　Deploy
<br>アクションプロバイダー　AmazonECS
<br>入力アーティファクト　BuildArtifact
<br>クラスター名　ecs-cluster
<br>サービス名　laravel-app-ecs
<br>イメージ定義ファイル　imagedefinitions.json
<br>完了を押し、　保存する　を選択
<br>

### 4. buildspec.ymlを修正し、Gitpush

* buildspec.ymlの末尾に以下を追記する
```
post_build:
  commands:
    ...略...
    - echo Writing image definitions file...
    - echo "[{\"name\":\"web\",\"imageUri\":\"${REPOSITORY_URI}:web\"},{\"name\":\"app\",\"imageUri\":\"${REPOSITORY_URI}:app\"}]" > imagedefinitions.json

artifacts:
    files: imagedefinitions.json
```

* 変更をGitにPUSHし、CodePipelineが動作することを確認
