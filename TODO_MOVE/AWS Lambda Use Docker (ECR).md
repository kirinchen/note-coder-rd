---
title: AWS Lambda Use Docker (ECR)
tags: [aws]

---

# AWS Lambda Use Docker (ECR)

## ECR

1. get-login-password

```bash
aws ecr get-login-password --region region | docker login --username AWS --password-stdin aws_account_id.dkr.ecr.region.amazonaws.com

aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin 420798909600.dkr.ecr.ap-southeast-1.amazonaws.com
```

> aws_account_id go to [Iamv2 Page](https://console.aws.amazon.com/iamv2/home?#/home)

2. Create a repository

```bash
aws ecr create-repository     --repository-name hello-world     --image-scanning-configuration scanOnPush=true     --region ap-southeast-1
```

3. To tag and push an image to Amazon ECR

```bash
docker tag hello-world:latest 420798909600.dkr.ecr.ap-southeast-1.amazonaws.com/hello-world:latest

docker push 420798909600.dkr.ecr.ap-southeast-1.amazonaws.com/hello-world:latest

```

## 參考文獻
* [使用 Python 建置 Lambda 函數](https://docs.aws.amazon.com/zh_tw/lambda/latest/dg/lambda-python.html)
* [Deploy Python Lambda functions with container images
](https://docs.aws.amazon.com/lambda/latest/dg/python-image.html)
* [Creating Lambda container images](https://docs.aws.amazon.com/lambda/latest/dg/images-create.html#images-create-from-base)
* [Using Amazon ECR with the AWS CLI](https://docs.aws.amazon.com/AmazonECR/latest/userguide/getting-started-cli.html)

###### tags: `aws`