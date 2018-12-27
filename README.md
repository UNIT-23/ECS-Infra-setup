# ECS Infrastructure Setup

Simple Nginx server running on amazon ECS EC2 instance

### Prerequisites

Make sure you have installed Docker and configured AWS cli 

* [AWS](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html) - AWS CLI SETUP
* [Docker](https://docs.docker.com/install/overview/) - DOCKER SETUP

### USAGE

A step by step guide that tells you how to get a running server on amazon ecs

Commands which are being used in cloudformation
```
    aws cloudformation create-stack --template-body file://$PWD/infra/vpc.yml --stack-name vpc

    aws cloudformation create-stack --template-body file://$PWD/infra/ecs-roles.yml --stack-name roles  --capabilities CAPABILITY_IAM

    aws cloudformation update-stack --template-body file://$PWD/infra/vpc.yml --stack-name vpc
    
    aws cloudformation describe-stacks --stack-name vpc
    
    aws cloudformation delete-stack  --stack-name vpc

```


Step 1) Push your image to ECR
```

docker build -t nginx-ecs .
docker run --name some-nginx -p 8080:80 nginx-ecs


-   Create repository in ecr
    CMD: aws ecr create-repository --repository-name nginx-ecs
    description: Creates repo in ECR
-   Login to ecr
    CMD:  aws ecr get-login --no-include-email | sh
    description: This command is used to run docker login with aws config so that we can use docker push
-   Push image to created repo
    CMD: IMAGE_REPO=$(aws ecr describe-repositories --repository-names nginx-ecs --query 'repositories[0].repositoryUri' --output text)
     
    docker tag nginx-ecs:latest $IMAGE_REPO:v1
    docker push $IMAGE_REPO:v1

```


Step 2) Create VPC Stack 
```
     aws cloudformation create-stack --template-body file://$PWD/infra/vpc.yml --stack-name vpc
```

Step 3) Create Roles which we will use for ecs and ec2
```
      aws cloudformation create-stack --template-body file://$PWD/infra/ecs-roles.yml --stack-name roles  --capabilities CAPABILITY_IAM
```

Step 4) Create loadbalancer, targetgroup, securitygroups and attach them to subnets created in vpc stack
```
     aws cloudformation create-stack --template-body file://$PWD/infra/loadbalancer.yml --stack-name loadbalancer
```

Step 5) Create taskDefinition for ecs service 
```
     aws cloudformation create-stack --template-body file://$PWD/infra/create-task-definition.yml --stack-name task
```

Step 6) Create service which will run our task
```
     aws cloudformation create-stack --template-body file://$PWD/infra/create-service.yml --stack-name service
```


That's it now you can go to public dns of your instance from aws console.


## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

