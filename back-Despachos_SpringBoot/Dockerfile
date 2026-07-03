
FROM maven:3.8.5-openjdk-17 AS build
WORKDIR /app
COPY . .

WORKDIR /app/Springboot-API-REST-DESPACHO
RUN mvn clean package -DskipTests


FROM eclipse-temurin:17-jre-alpine
WORKDIR /app

COPY --from=build /app/Springboot-API-REST-DESPACHO/target/*.jar app.jar

RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

EXPOSE 8082
ENTRYPOINT ["java", "-jar", "app.jar"]