FROM maven:3.9-eclipse-temurin-11 AS build
WORKDIR /app
COPY . .
RUN mvn clean package -DskipTests

FROM eclipse-temurin:11-jre
WORKDIR /app
COPY --from=build /app/target/common-data-service.jar app.jar

EXPOSE 9000
ENTRYPOINT ["java", "-jar", "app.jar"]