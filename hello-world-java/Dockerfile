# Build a JAR File
#FROM maven:3.8.2-jdk-8-slim AS stage1
#WORKDIR /home/app
#COPY . /home/app/
#RUN mvn -f /home/app/pom.xml clean package

# Create an Image
#FROM openjdk:8-jdk-alpine
#EXPOSE 5000
#COPY --from=stage1 /home/app/target/hello-world-java.jar hello-world-java.jar
#ENTRYPOINT ["sh", "-c", "java -jar /hello-world-java.jar"]


# ---- Stage 1: Build the Spring Boot JAR ----
FROM maven:3.9.6-eclipse-temurin-17 AS builder

# Accept proxy settings as build args (optional)
ARG HTTP_PROXY=http.docker.internal:3128
ARG HTTPS_PROXY=http.docker.internal:3128

# Set proxy environment variables
ENV http_proxy=${HTTP_PROXY}
ENV https_proxy=${HTTPS_PROXY}

# Set working directory
WORKDIR /home/app

# Copy only pom.xml first to leverage Docker layer caching
COPY pom.xml .

# Install certificates (important for HTTPS Maven repos)
RUN apt-get update && apt-get install -y ca-certificates && update-ca-certificates

# Optional: copy Maven proxy config if present
COPY settings.xml /root/.m2/settings.xml

# Pre-download dependencies with retry logic
RUN mvn dependency:go-offline -B -Dmaven.wagon.http.retryHandler.count=3

# Copy source code
COPY src ./src

# Build the JAR file
RUN mvn clean package -DskipTests

# ---- Stage 2: Create the runtime image ----
FROM eclipse-temurin:17-jre-alpine

# Create app directory
WORKDIR /app

# Copy built JAR from builder stage
COPY --from=builder /home/app/target/*.jar app.jar

# Expose port (change if your app uses another)
EXPOSE 5000

# Start the app
ENTRYPOINT ["java", "-jar", "app.jar"]
