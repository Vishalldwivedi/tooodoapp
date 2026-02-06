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

  
