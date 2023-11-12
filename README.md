### コンテナベースのデプロイ、CI/CD パイプラインの構築のデモ  

## ReactアプリをDocker composeでECSでデプロイする
## 前提条件
- AWS CLIがインストールされていること
- AWS IAMにて適切なユーザーが作成済みであること（AmazonEC2ContainerRegistryPowerUserロールが必須です）
- Docker Desktopがインストールされていること
- Nodeがインストールされていること

-----


## Docker CLI認証
Docker CLIを認証をする。Amazon ECRを使用してイメージをpushおよびpullが可能  
```aws ecr get-login-password --region <リージョン名> | docker login --username AWS --password-stdin <AWSアカウントID>.dkr.ecr.<リージョン名>.amazonaws.com```

## AWS ECR(Elastic Container Registry)にリポジトリを作成する
Dockerイメージを保存するためのリポジトリを作成。aws-react-ecsという名前でリポジトリを作成  
```aws ecr create-repository \ --repository-name aws-react-ecs \ --image-scanning-configuration scanOnPush=true \ --region <リージョン名>```  
<img width="469" alt="スクリーンショット 2023-11-12 9 56 22" src="https://github.com/Kana-Karin/aws-ecs-react/assets/84316229/b250b399-f9f5-4e89-b4ae-a75e3abe6ba9">


実際にAWS ECRコンソール画面に行き、リポジトリが作成されているか確認します。  
<img width="401" alt="スクリーンショット 2023-11-12 10 03 04" src="https://github.com/Kana-Karin/aws-ecs-react/assets/84316229/ae1a5f4e-c958-4ac7-928e-020d2fa41108">


## Reactプロジェクトの作成
create-react-appコマンドで生成。  
```npx create-react-app aws-react-ecs --template typescript```
```cd aws-react-ecs```

## デプロイファイルの作成
```touch Dockerfile docker-compose.yaml```

## Dockerイメージをプッシュする
DockerfileからDockerイメージをビルドし、タグをつけます。  
```docker build -t <aws_account_id>.dkr.ecr.<region>.amazonaws.com/aws-react-ecs .```

## ECRにDockerイメージをプッシュ
```docker push <aws_account_id>.dkr.ecr.<region>.amazonaws.com/aws-react-ecs:latest```  
  
イメージがプッシュされているか、ECRコンソール画面を確認します。  
<img width="426" alt="スクリーンショット 2023-11-12 10 58 27" src="https://github.com/Kana-Karin/aws-ecs-react/assets/84316229/4937edaf-7e95-4480-b400-457cfba66121">


## ECS contextの作成
```docker context create ecs <react-ecs-sample>```  
react-ecs-sampleという名前のECS Dockerコンテキストを作成します。  
AWS CLIをインストールしているので、Amazonに接続するための既存のAWSプロファイルを選択します。  

## 作成したコンテキストを使用
```docker context use <react-ecs-sample>```  

## docker composeでECSへデプロイ
```docker compose up```  
暫くの間ECSへのデプロイに時間がかかります。~~この間が一番ドキドキします・・・~~  
10分ほど待機し、無事にデプロイ完了しました!!!  
![スクリーンショット 2023-11-12 11 21 20](https://github.com/Kana-Karin/aws-ecs-react/assets/84316229/8457854b-8b80-4753-a9bd-6835d183b4d9)


ECSコンソール画面に行って確認します。  
<img width="400" alt="スクリーンショット 2023-11-12 11 49 19" src="https://github.com/Kana-Karin/aws-ecs-react/assets/84316229/a311150f-fe68-4b7e-b4ac-8ebfc9196c1c">


実際にデプロイが出来ているか確認  
EC2→Load balancersからDNSを確認し、リソースにアクセスします。  
<img width="438" alt="スクリーンショット 2023-11-12 11 57 47" src="https://github.com/Kana-Karin/aws-ecs-react/assets/84316229/7a1f7f71-6fc3-4662-b2ef-b673272aea5f">  

実際にアクセスをしReact Projectの画面が確認できました！！  


### 最後にリソースの削除を行います
- LoadBalancer
- CloudFormation
- ECR
- ECS
- S3
- VPC

## つまづいた部分
- compose upの際認証情報で誤った認証情報を入れていたので、現在のコンテキストを削除をし、  
  ```docker context rm aws-react-ecs -f```を実行  
  新しい認証情報でコンテキストを作成する```docker context create ecs aws-react-ecs```
- Node.js14の情報を参照していたので、最新のNode.js18でイメージを作成、ビルド
- docker compose upを実行した際、Attaching to のままハングアウトしていたので、docker ps、docker logsで原因を確認 context認証情報を再生成した際、docker context use実行をしていませんでした。

## 参照させていただいたサイト
[Docker 公式サイト - Deploying Docker containers on ECS](https://docs.docker.com/cloud/ecs-integration/)  
[AWS公式サイト - CLI での Amazon ECR の使用](https://docs.aws.amazon.com/ja_jp/AmazonECR/latest/userguide/getting-started-cli.html)  
[AWS ECRとECSの入門(EC2編) ~ ECSのEC2版を使ってReactのDockerアプリケーションをAWS上で稼働させる方法 ~](https://casualdevelopers.com/tech-tips/how-to-deploy-dockerized-react-application-to-aws-with-ecr-and-ecs-ec2/)  
