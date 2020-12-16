
No1.Codebuild編=====

git clone する
docker-compse build
docker imagesで　ecs-webとecs-appのlatestを見る

ECR作成

マネコンでプッシュコマンドの表示
aws ecr get-login-password --region ap-northeast-1 | docker login --username AWS --password-stdin 551419436295.dkr.ecr.ap-northeast-1.amazonaws.com
＞Login Succeeded

tagづけ
docker tag ecs_web:latest 551419436295.dkr.ecr.ap-northeast-1.amazonaws.com/ecs:web
docker tag ecs_app:latest 551419436295.dkr.ecr.ap-northeast-1.amazonaws.com/ecs:app

upload
docker push 551419436295.dkr.ecr.ap-northeast-1.amazonaws.com/ecs:web
docker push 551419436295.dkr.ecr.ap-northeast-1.amazonaws.com/ecs:app

upload完了

Cluster
 EC2 linux ネットワーキング 
 名　ecs-cluster
 t2.micro
 KEY ec-app（オプション）
 　VPC
 　サブネット（パブリック）
 　SecurityGroup

No2.CodePipelineでDeployまで=====

RDS用意できている
　DB_HOST RDSエンドポイント 
  DB_NAME laravel
　DB_USERNAME RDSユーザー名
　DB_PASSWORD RDSパスワード 

ECS Cluster作成
　EC2 Linux + ネットワーキング
　ecs-pipeline-cluster
　ｔ２，２台
　キー：オプション
　サブネット：Pub1
　セキュリティ：いつもの
　作成

　ECSインスタンスが2台あがっていること

タスク定義
　新しいタスク定義
　EC2を選択
　ecs-pipeline-task
  ecsTaskExecutionRole
  ネットワークdefault
  タスク定義
　新しいタスク定義
　コンテナ追加
　　web
　　　イメージ　551419436295.dkr.ecr.ap-northeast-1.amazonaws.com/ecs:web
　　　memory 300
　　　ポート 80:80/tcp
　　　リンク app:app
　　app
　　　イメージ　551419436295.dkr.ecr.ap-northeast-1.amazonaws.com/ecs:app
　　　memory 600
　　　環境変数
　　　DB_HOST
　　　DB_NAME
　　　DB_＿_PASSWORD
　作成

そのまま、アクション→サービス作成

　起動タイプ　EC2
　サービス名　larave-app-ecs
　REPLICA、タスクの数１
　ローリング
　作成

タスクからインスタンスのIPアドレスをチェックし
まずはアクセスしてLaravel画面が出るのを見る

PIPELINEの作成

前に作っているecs-pipelineを選択する

ステージ追加、Deploy
アクショングループ追加　Deploy、プロバイダーECS、アジア、アーティファクトはBuild、クラスター・サービス選択、イメージ定義ファイル　imagedefinitions.json　で完了し保存

