
# stage first maven base image 
FROM maven:3.9.6-eclipse-temurin-21 AS build
WORKDIR /app

COPY pom.xml .
RUN mvn dependency:go-offline -q

COPY src ./src
RUN mvn clean package -DskipTests


# stage 2 distroless image 
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
# created a appgroup and add appuser to appgroup and use appuser 
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

#just copy the jar in this stage 
COPY --from=build /app/target/*.jar app.jar 

EXPOSE 8080

ENV SPRING_PROFILES_ACTIVE=final
ENV SPRING_DATASOURCE_URL=jdbc:mysql://host.docker.internal:3306/todoDB?createDatabaseIfNotExist=true
ENV SPRING_DATASOURCE_USERNAME=vishal
ENV SPRING_DATASOURCE_PASSWORD=dwivedi


ENTRYPOINT ["java", "-jar", "app.jar"]
