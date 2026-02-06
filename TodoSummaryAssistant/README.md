1. Project Understanding & Local Execution

spring.datasource.url=jdbc:mysql://localhost:3306/todo_db?createDatabaseIfNotExist=true
spring.datasource.username=root
spring.datasource.password= my pass
cohere.api.key= my token 
slack.webhook.url= my webhook 

mvn clean install
mvn spring-boot:run
npm install
npm start 
backend http://localhost:8080 , frontend -> http://localhost:3000

Problem faced setting up mysql db ->  solved via running mysql as container 

2. Containerization (Docker)

first run my sql DB -> docker run -d \
  --name mysql-db \
  -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=dwivedi \
  -e MYSQL_USER=vishal \
  -e MYSQL_PASSWORD=dwivedi \
  -e MYSQL_DATABASE=todoDB \
  mysql:8

then todo app -> docker run -d \
  --name todo-app \
  -p 8080:8080 \
  todo-app:final

  Design pattern of docker file -> 

Stage 1 uses Maven to compile and package the app.

Stage 2 uses Alpine JRE image for smaller and faster deploy 

non root user -> 
We create appuser and run the app as non‑root → better security for production.

Externalize configuration -> 
DB URL, username, password, and profile are passed via environment variables and not hardcoded.
You can change them per environment (dev/test/prod) without rebuilding the image.

only the final small image size is used no pom.xml or src in final image 

Front end app -> 
 docker build -t todo-frontend:prod .

 docker run -d \
  --name todo-frontend \
  -p 3000:80 \
  todo-frontend:prod

  
3. JenkinsFile setup -> 

Pipeline Overview

The pipeline automates the following stages:

Checkout

-> Pulls the code from GitHub repository 
-> Uses GitHub Webhook to trigger pipeline on code push 
-> Build & Docker Image
-> Pushes the image to DockerHub using stored credentials (username & access token).
-> Code Quality & Security Scanning
-> SonarQube: Runs code quality analysis and reports bugs, code smells, and vulnerabilities.
-> Trivy: Performs container image vulnerability scanning.
-> OWASP Dependency Check: Scans project dependencies for known vulnerabilities.

DEPLOYMENT USING GITOPS 

-> Updates Kubernetes manifests in the GitOps repo 
-> ArgoCD syncs the changes automatically to the EKS  . 


_REFERENCES -> _

Shared Library: https://github.com/Vishalldwivedi/JenkinsSharedLib

GitOps Repo: https://github.com/Vishalldwivedi/gitopsTodoApp


4. KUBERNETES MENEFESTS -> 

k8s setup-> in EKS in master machine 
A.create access and secret access keys for aws cli -> aws configure 
B. install eksctl and kubectl
C. create eks cluster  -> eksctl create cluster -name - region - version // takes 20 mins 
D. install trivy 
E . run sonarQube as docker container // user pass as admin admin 


https://github.com/Vishalldwivedi/gitopsTodoApp

backend , frontend pods + services attached to at for service discoverly and loadbalancing using labes and selectors so that pods can talk to each other 



5. GITOPS DEPLOY  -> 
argoCd setup steps -> 
A . kubectl create namespace argoCD  , kubectl apply -n argo 
B. install argoCD cli 
C. kubectl get svc -n arogcd (default is clusterIP change it to Nodeport)
D. from master node to k8s cluster this argoCD is running 
E. open security groups of k8s node where argoCd is run 
F. access ARgo cd UI uisg publicIP:NodePort 
G. user name and pass is admin and kubectl - n argoCD get secrets and then add giops repo in argo CD ui 

Deployment Flow -> 

Developer pushes code → Jenkins triggers CI pipeline via github webhook .
Jenkins builds Docker image → pushes to DockerHub via credentials 
Jenkins updates k8s/deployment.yaml in gitopsTodoApp with new image tag → commits & pushes via gitops 
ArgoCD detects Git change → applies updated manifests to EKS cluster .
App is updated in EKS 

Example Rollback Flow -> 
When a faulty deployment is detected
Revert the commit in Git that updated the image tag.
ArgoCD detects the reverted commit → syncs the cluster back to the previous working version.
ArgoCD auto matick autoSync to github . 


6. AWS EKS Setup via console and also terraform + S3 and dynamoDB for remote backend

A. aws configure to connect terraform with aws 
B. Follow modules for better collaboration and reusable code 
C. create 2 differnet modules for vpc and eks 
D. vpc module contain vpc + public private subnets + each subnet associated with route table for IGW and NAT gate way access 
E. for EKS cluster i have create a aws_iam_role + aws_iam_role_policy_attachment and also aws_iam_role + aws_eks_cluster and for worker node -> aws_iam_role + aws_iam_role_policy_attachment + aws_eks_node_group
D. Finally setup S3 for remote backend and dynamodb for state locking in AWS . 

terraform EKS + VPC + S3 setup github repo -> https://github.com/Vishalldwivedi/EKS-Terraform-todoapp/tree/main

7. 


