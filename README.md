<<<<<<< HEAD
# **Docker-ECR-Codebuild**

## **About**

gitレポジトリーの変更をトリガーにECRへのPUSH、およびCodebuildをする
=======
# **ECS-CodeDeploy**

<br>

## **About**

先程作成したCodebuildにECSでのDeployフェーズを追加する

<br>
>>>>>>> 52151e82f7cc1dd87dcfd5dc9f89c60ed6bff767

## **Inside Package**
 * web nginx:alpine
 * app php
 * DBはAWSのRDSを使用

<<<<<<< HEAD
## **手順**

### **1. ECRレポジトリの作成**

* AWSマネコンで　ECR > リポジトリ > リポジトリを作成
```
 可視性設定　プライベート
 リポジトリ名　Laravel-app-ecs(例)
 リポジトリを作成
```
=======
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
<br>環境変数　以下のRDS接続パラメータを設定
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

* 変更をGitにPUSHし、Soucrce->Build->Deployの3段階でCodePipelineが動作することを確認


* アプリへのアクセス
<br>
<br>クラスター > ecs-cluster の詳細画面を表示
<br>タスク タブで実行中のコンテナインスタンスを選択
<br>パブリックIPを確認しブラウザでアクセス -> Laravelアプリが表示されること

* 初回のDB Migration
<br>
<br>クラスターにSSHアクセスし、docker exec -it <CONTAINER ID> bash
<br>php artisan migrate
>>>>>>> 52151e82f7cc1dd87dcfd5dc9f89c60ed6bff767

### **2. 「プッシュコマンドの表示」をクリックし、ポップアップの内容に従いローカルターミナルで以下**

* Docker クライアントを認証
```
aws ecr get-login-password --region ap-northeast-1 | docker login --username AWS --password-stdin xxxxxxxxxxxx.dkr.ecr.ap-northeast-1.amazonaws.com
...

Login Succeeded　←成功
```

* イメージを構築
```
docker-compose build
```

* イメージにタグ付け
docker images コマンドで ecs-pipeline__webと、ecs-pipeline__appの2つがビルドされていることを確認し、それぞれのイメージにタグ付けをする

```
docker images
 REPOSITORY        TAG 
 ecs-pipeline_web  latest
 ecs-pipeline_app  latest

docker tag ecs-pipeline_web:latest xxxxxxxxxxxx.dkr.ecr.ap-northeast-1.amazonaws.com/laravel-app-ecs:web
docker tag ecs-pipeline_app:latest xxxxxxxxxxxx.dkr.ecr.ap-northeast-1.amazonaws.com/laravel-app-ecs:app
```

* イメージをECRリポジトリにPUSH

```
docker push xxxxxxxxxxxx.dkr.ecr.ap-northeast-1.amazonaws.com/laravel-app-ecs:web
docker push xxxxxxxxxxxx.dkr.ecr.ap-northeast-1.amazonaws.com/laravel-app-ecs:app
```

* 確認
<br>AWSマネコンで ECR > リポジトリ > laravel-app-ecs（例）でweb, appの2つのイメージが保存されていることを確認する


### 3. CodeBuildの作成

* AWSマネコンで　デベロッパー用ツール > CodeBuild > ビルドプロジェクト > ビルドプロジェクトを作成
```
プロジェクトの設定
  プロジェクト名　laravel-app-ecs-build（例）

ソース
  ソースプロバイダ　Github
  リポジトリ　OAuth を使用して接続する　を選択
  Githubに接続　をクリック
  ポップアップで　確認　を選択
  GitHub リポジトリ　https://github.com/siwai0208/ecs-pipeline.git　を選択

環境
  環境イメージ　マネージド型イメージ
  オペレーティングシステム　Amazon Linux 2
  ランタイム　Standard
  イメージ　aws/codebuild/amazonlinux2-x86-64-standard:3.0
  特権付与　有効化する
  サービスロール　新しいサービスロールを選択
  追加設定　環境変数に以下の値を追加
    AWS_ACCOUNT_ID
    AWS_DEFAULT_REGION
    DOCKERHUB_USER
    DOCKERHUB_PASS
    IMAGE_REPO_NAME

ビルドプロジェクトを作成する
```

### 4. CodePieplineの作成

* AWSマネコンで　デベロッパー用ツール > CodePipeline > パイプライン > パイプラインを作成する
```
パイプラインの設定
  パイプライン名　laravel-app-ecs-pipeline（例）
  高度な設定　laravel-app-image（例）

ソースステージを追加する
  ソースプロバイダー　Github(バージョン2)
  接続　Githubに接続する
  接続名　food-app-ecs-pipeline
  GitHub アプリ　「3. CodeBuildの作成」で作成したアプリを選択し接続
  リポジトリ名　siwai0208/ecs-pipeline
  ブランチ名　main

ビルドステージを追加する
  プロバイダーを構築する　AWS CodeBuild
  プロジェクト名　laravel-app-ecs-build（3. CodeBuildの作成で作成）
  次に

デプロイ
  導入段階をスキップ
  パイプラインを作成する
```

### 5. CodePipelineのテスト

* パイプラインを作成後、自動でソースチェック→BuildがスタートするがBuildフェーズで失敗となる。

* AWSマネコン　IAM > ロール で「3. CodeBuildの作成」で作成した新しいサービスロールを選択し「AmazonEC2ContainerRegistryFullAccess」「AmazonS3FullAccess」をポリシーをアタッチ

* Codepipelineに戻り、Buildを再試行
