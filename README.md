# Deploy a Serverless Golang App to AWS Fargate

<img src="https://d1.awsstatic.com/product-page-diagram_Amazon-ECS%402x.0d872eb6fb782ddc733a27d2bb9db795fed71185.png">



## Pre-Requirements
* Docker
* Golang
* AWS Account
* AWS CLI


## Step 1

Create a golang project with main.go file. Paste the below code.
```bash
package main

import "github.com/gofiber/fiber/v2"

func main() {
	app := fiber.New()

	app.Get("/", func(c *fiber.Ctx) error {
		return c.SendString("Hello, World!")
	})

	app.Listen(":80")
}
```

## Add fiber framework
```bash
go get github.com/gofiber/fiber/v2
```
## Step 2

Create a Dockerfile. Paste the below code.

```bash
FROM golang:1.16-alpine
WORKDIR /app
COPY go.mod .
COPY go.sum .
RUN go mod download
COPY . .
RUN go build -o ./out/dist .
CMD ./out/dist
```

Run the below command on the terminal to generate out and dist.
```bash
go build -o ./out/dist .
```

## Step 3
Build Docker container.
```bash
docker build -t app .
```

Run Docker container.

```bash
docker run -p 8888:80 app
```
If you open http://127.0.0.1:8000 or http://localhost:8888 on your browser, you will see the output.

## Step 4
Let's push the project to the cloud. Open your AWS Management Console.

### Elastic Container Registery

* Create a repository with name of app either private or public. In my case, I choose public. We will push Docker container to this repository. 


After created repository, you will see Push commands for app.

* Retrieve an authentication token and authenticate your Docker client to your registry.
  Use the AWS CLI:
```bash
aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/c0r3n4u9
```
Here, I got an error. (AWS ECR user is not authorized to perform: ecr-public:GetAuthorizationToken on resource:)
If you got same error, you can add permision. (AmazonElasticContainerRegistryPublicFullAccess)
Solution : https://stackoverflow.com/questions/65727113/aws-ecr-user-is-not-authorized-to-perform-ecr-publicgetauthorizationtoken-on-r
* (Because we already build docker container, you can skip this command.
  )
Build your Docker image using the following command. For information on building a Docker file from scratch see the instructions here . You can skip this step if your image is already built:
```bash
docker build -t app .
```

* After the build completes, tag your image so you can push the image to this repository:

```bash 
docker tag app:latest public.ecr.aws/c0r3n4u9/app:latestdocker run -p 8888:80 app
```

* Run the following command to push this image to your newly created AWS repository:

```bash
docker push public.ecr.aws/c0r3n4u9/app:latest
```

### Amazon Elastic Container Service (ECS)

* Copy URI of the created repository. 
* Create a cluster. 
* Select template as Networking only for AWS Fargate.
* Type name of cluster.
* Select create default VPC.
* After cluster is created, click view cluster and task definitions.
* Create a new task definition.
* Select Fargate.
* Type name for task definition.
* Task role will be none.
* Task Memory is 0.5 GB and Task CPU is 0.25.
* Add container. Type a name for your container. Paste the copied URI to Image part.
* Set the port as 80.

After created cluster, we can create our service.
* Select launch type as Fargate.
* Select task definition that we created.
* Type a service name.
* Number of tasks : 1 (one container, you can select more.)
* Click next step.
* Select one of subnet option.
* Load-balancer will be none in our project.
* Click next step.
* We don't need auto-scaling.
* Click create service.
* After created service, task is running and it gives us a public IP.
* Paste the IP address to your browser, you will see the app we created.

## Watch video on Youtube.
(https://www.youtube.com/watch?v=yefgBi9j0w0&t=384s)

