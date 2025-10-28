https://github.com/davidmoten/maven-demo.git
#only change jar to war in pom.xml 
#build command
#mvn clean package
pipeline {
    agent any

    tools {
        maven '3.9.11'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', 
                    url: 'https://github.com/athrv1837/JenkinsBuildTest.git'
            }
        }

        stage('Build WAR') {
            steps {
                bat 'mvn -f my-webapp/pom.xml clean package'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                deploy adapters: [tomcat9(
                    credentialsId: 'tomcat-credentials',
                    url: 'http://localhost:9009'
                )],
                war: 'my-webapp/target/my-webapp.war'
            }
        }
    }

    post {
        success {
            echo '✅ Build and Deploy Successful!'
        }
        failure {
            echo '❌ Build or Deploy Failed!'
        }
    }
}

#10 Repo
https://github.com/ArpitaKadtane/mydockerapp.git

#Jenkins cred 
Auth token - : 09cebf21883a4b2c9579fc95c902182a
username - : admin 

#Pipeline for any java application 
https://github.com/athrv1837/myjavaapp.git

#dockerfile for springboot
# -------------------------------------------------
# 1. Build stage – compile & package the app
# -------------------------------------------------
FROM eclipse-temurin:21-jdk-alpine AS builder
WORKDIR /app

# Copy Maven/Gradle wrapper + pom/build files
COPY pom.xml .                 # Maven
# COPY build.gradle settings.gradle .  # Uncomment for Gradle
COPY src ./src

# Build the JAR (skip tests for faster CI, remove -DskipTests if you want tests)
RUN ./mvnw clean package -DskipTests   # Maven
# RUN ./gradlew bootJar                 # Gradle (uncomment if using Gradle)

# -------------------------------------------------
# 2. Runtime stage – tiny image with only JRE
# -------------------------------------------------
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app

# Copy the built JAR from the builder stage
COPY --from=builder /app/target/*.jar app.jar

# Expose the default Spring Boot port
EXPOSE 8080

# Run with explicit JVM opts (optional but recommended)
ENTRYPOINT ["java", "-XX:+UseContainerSupport", "-Djava.security.egd=file:/dev/./urandom", "-jar", "/app/app.jar"]

# From the project root (where Dockerfile lives)
docker build -t yourdockerhubusername/my-springboot-app:latest .

docker run -d -p 8080:8080 --name myapp yourdockerhubusername/my-springboot-app:latest
docker stop myapp && docker rm myapp
docker login
# → enter your Docker Hub username + password (or use a personal access token)
docker tag yourdockerhubusername/my-springboot-app:latest \
           yourdockerhubusername/my-springboot-app:v1.0.0
docker push yourdockerhubusername/my-springboot-app:latest
docker push yourdockerhubusername/my-springboot-app:v1.0.0
