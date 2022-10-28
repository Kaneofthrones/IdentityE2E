# Identity E2E Infrastructure Demo

dentityE2E has been identified by a client to migrate an existing on-premises service from a datacentre to the cloud. The client sees this migration as an avenue for reducing costs and increasing their service reliability as their existing service is suffering from outages due to hardware failures. They would also like to take this opportunity to increase their security posture due to several high-profile news items relating to information security. The client does not have experience working with cloud platforms and has approached you asking you implement a pilot solution.
The environment will be extended in the future as their services are migrated. The client has indicated that they would rather use AWS over a platform as a service (Heroku, AppEngine, etc) as this would facilitate their integration with other Cloud-based partners.

## Task 

Using infrastructure as code, implement a solution which will deploy the attached microservices to a functional state. The backend must be able to communicate with DynamoDB, and the front end must be able to communicate with the backend.

## Considerations

	•	The client intends to migrate more applications in the future. How will the solution remain intuitive to new developers and scale to their needs?
	•	Encrypting resources at rest and in transit will help the client improve their security position.
	•	Enabling services to be self-recovering will reduce staff time spent on-call.
	•	How can the application be deployed in such a way to scale with demand?
	
##README Begins Here
	
Requirements / Tools Used:
	
* awscli
* aws credentials saved in ~/.aws/credentials | configured
* docker
* terraform
* kubectl
* aws cdk
* python 

### 1. Create branches using git flow in mind

Main -> Dev -> Features -> Dev -> Main

![image](https://user-images.githubusercontent.com/30622959/141043288-7785614f-9d44-43b1-a47f-ffb835756e71.png)

### 2. Create infrastructure on AWS with IaC tools: Terraform & AWS CDK

There are multiple approaches availble and as this task requires only provisioning and not configuration management terraform is a strong choice but also the new code agnostic AWS CDK is a strong option and can help with multi-talented teams if this solution was to be scaled up so i have included little examples of both

All terraform code can be found in the /terraform directory 

This code will provision a VPC, security groups, Iam permisisons and a DynamoDB Table 

### 3. Package app into docker images

build the Dockerfiles in both and frontend and backend folders of the app directory and tag with the appriate aws account number and distination prefix so they can then be pushed to ECR on AWS 
 ```docker tag flask-docker-frontend-app:latest <aws_account number>.dkr.ecr.eu-west-2.amazonaws.com/flask-docker-frontend-app:latest
 
 docker push <aws_account number>.dkr.ecr.eu-west-2.amazonaws.com/flask-docker-frontend-app:latest```

### 4. Test locally

To avoid AWS charges it important to test the apps locally with a local DynamoDB instance

Make a docker compose file and add the following to the services 

``` 
  dynamodb:
    image:  amazon/dynamodb-local
    container_name: my-dynamodb
    hostname: dynamodb
    restart: always
    volumes:
      -  ./my-dynamodb-data:/home/dynamodblocal/data
    ports:
      - 8000:8000
    command: "-jar DynamoDBLocal.jar -sharedDb -dbPath /home/dynamodblocal/data/" 
```    
   You should be able to access the service client with the following:
    
    db = boto3.client('dynamodb',endpoint_url='http://localhost:8000')

To view with a GUI and create Tables use dynamodb-admin

``` 
    npm install -g dynamodb-admin 
    // For Mac/Linux
    DYNAMO_ENDPOINT=http://localhost:8000 dynamodb-admin

```
  
To test the apps locally run:
``` docker compose up```

![image](https://user-images.githubusercontent.com/30622959/198674388-e5de63ff-5b6c-4975-b29f-4c1e4eb1f5a4.png)

If any changes need to be made, make a new image and push to ECR

### 5. K8s future

For future reference you can deploy the apps on ECS first then EKS, in the AWS_CDK folder there will code to do this however to avoid costs this should not be tested 

Ideally the Microservices will be on an AWS kubernetes cluster managed by terraform or AWSCDK with the backend and front end in different subnets, all traffic should go through a load balancer (like in the terraform code) and then hit the frontend which will then interact with the backend Also by switching to this solution the self healing properties of K8s should reduce time spent on support calls   

### 6. Migration and more future Notes

To migrate from a monolith to a microservices approach requires the following steps:

Containerize the Monolith -> Deploy the Monolith (on AWS ECS) -> Break the Monolith -> Deploy Microservices

AWS CDK should remain intuitive for new developers as multiple coding languages are available, also there is a CDK for terraform cdktf which could also attract a wider skillset moving forward



