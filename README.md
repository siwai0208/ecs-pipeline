# **Docker-ECS-Codepipeline**

## **About**

[ECR-Codebuild](https://github.com/siwai0208/ecr-codebuild) をベースに、ECSでのDeployを追加
  GitへのPUSHをトリガーに、CodebuildでECRイメージを更新し、更新されたイメージでECSを起動させる

## **Inside Package**
 * docker-compose.yml
     web nginx:alpine
     app php-fpm
     DBはAWSのRDSを使用
 * buildspec.yml
     ECR login→build→tagづけ→ECRへpush→imagedefinitions.jsonを作成

## **手順**

### 0. RDS構築済みであること

[Cloudformation-RDS](https://github.com/siwai0208/cloudformation/tree/main/rds) で RDSを準備しておく

### 1. ECS Clusterの作成

* Amazon ECS > クラスター > クラスターの作成
```
 クラスターテンプレート
   EC2 Linux + ネットワーキング
  
 クラスターの設定
   クラスター名　ecs-cluster（例）
  
 インスタンスの設定
   EC2 インスタンスタイプ　t2.micro
   インスタンス数　2 ※t2.micro 1台ではDeployのメモリ不足
   キーペア　自身のSSH-KEY
  
 ネットワーキング
   VPC　既存VPCを選択、RDSと同一VPCであること
   サブネット　パブリック
   セキュリティグループ　WEB・SSHの許可
```

* 確認
  クラスター > ecs-cluster　のステータス画面で ECSインスタンス > コンテナインスタンス が2つあること

### 2. タスク定義の作成

* Amazon ECS > タスク定義 > 新しいタスク定義の作成
```
 起動タイプ　EC2
  
 タスクとコンテナの定義の設定
   タスク定義名　ecs-cluster-task
   タスクロール　ecsTaskExecutionRole
  
 コンテナの定義 > コンテナの追加
  
 webコンテナの設定
  スタンダード
   コンテナ名　web
   イメージ  ECRレポジトリのwebイメージのURIをコピペ
   メモリ制限 (MiB)*　ハード制限・300
   ポートマッピング　80:80
   ネットワーク設定 リンク app:app
  追加
  
 appコンテナの設定
  スタンダード
   コンテナ名　app
   イメージ  ECRレポジトリのappイメージのURIをコピペ
   メモリ制限 (MiB)*　ハード制限・600
   環境  環境変数　以下のRDS接続パラメータを設定
   DB_HOST　Value　RDSのエンドポイントをコピペ
   DB_NAME　Value　RDSで作成したデータベース名
   DB_USER　Value　RDSで作成したユーザー名
   DB_PASSWORD　Value　RDSで作成したパスワード
  追加

 作成
```

* 続けてサービスの作成　アクション > サービスの作成
```
 起動タイプ　EC2
 サービス名　laravel-app-ecs（例）
 サービスタイプ　REPLICA
 タスクの数　1
 デプロイメントタイプ　ローリングアップデート
  
 次のステップで画面を進め、サービスの作成
``` 

* 確認
  クラスター > ecs-cluster　のステータス画面で
<br>実行中のタスクの数 1個のEC2 であること
<br>タスク > コンテナインスタンス をクリックし、表示されるパブリックIPでアプリアクセス

### 3. CodePipelineの作成

* デベロッパー用ツール > CodePipeline > パイプライン<br>
  前回作成した[Codebuild](https://github.com/siwai0208/ecr-codebuild)のパイプラインを選択 > パイプラインをクローンする
``` 
  パイプライン名  ecs-codepipeline
  サービスロール　既存のサービスロール
  ロールのARN　Codebuildで作成したロールを選択
  バケット　Codebuildで指定したロールを選択
  クローンする
```
* Source(Gitレポジトリ)の変更
```
  編集 > Source > ステージを編集する
  GitHubに接続する
  接続名 ecs-codepipeline
  新しいアプリをインストールする
  Repository access > Only select repositories > siwai0208/ecs-pipelineを追加
  リポジトリ名 siwai0208/ecs-pipeline
  ブランチ名 main
```
* Deployの追加
```
  編集:Buildの下で　＋ステージを追加する　をクリック
  ステージ名　Deploy
  ＋アクショングループを追加する　をクリック
  アクション名　Deploy
  アクションプロバイダー　AmazonECS
  入力アーティファクト　BuildArtifact
  クラスター名　ecs-cluster
  サービス名　laravel-app-ecs
  イメージ定義ファイル　imagedefinitions.json
  完了
```
* 保存する

### 4. GitPushからの動作テスト

* 何かしらGitでPUSHし、Soucrce->Build->Deployの3段階でCodePipelineが動作することを確認

* アプリへのアクセス
  クラスター > ecs-cluster の詳細画面を表示
  タスク タブで実行中のコンテナインスタンスを選択
  パブリックIPを確認しブラウザでアクセス -> Laravelアプリが表示されること

* 初回のDB Migration
  クラスターにSSHアクセスし、docker exec -it <CONTAINER ID> bash
  php artisan migrate
  php artisan db:seed
