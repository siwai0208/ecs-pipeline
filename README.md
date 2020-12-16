
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


デベロッパー用ツール > CodePipeline > パイプライン > パイプラインを作成する

名　ecs-pipeline-3stage

ソース
Git2、ecs-pipeline-3stage接続する、アプリ選択して接続