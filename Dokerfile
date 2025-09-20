# Étape 1 : Build de l’application
FROM maven:3.8.5-openjdk-17 AS builder
WORKDIR /app

# Copier les sources et pom.xml
COPY pom.xml .
RUN mvn dependency:go-offline -B

COPY src ./src

# Compiler et packager l’application
RUN mvn clean package -DskipTests

# Étape 2 : Image finale
FROM openjdk:17-jdk-slim
WORKDIR /app

# Copier le JAR depuis l’étape précédente
COPY --from=builder /app/target/demo-0.0.1-SNAPSHOT.jar app.jar

# Exposer le port de l'application Spring Boot
EXPOSE 8080

# Lancer l’application
ENTRYPOINT ["java", "-jar", "app.jar"]
