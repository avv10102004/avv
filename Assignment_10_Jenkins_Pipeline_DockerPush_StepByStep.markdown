# Assignment 10: Jenkins Pipeline to Build and Push Docker Image - Step-by-Step Guide for Novices

## Prerequisites
- Maven, Jenkins, and Docker Desktop installed on Windows.
- A Docker Hub account (create at hub.docker.com).
- Git installed (for pushing to GitHub).

## Step 1: Prepare the Java Application
1. Open Command Prompt or PowerShell.
2. Create a project directory: `mkdir mydockerapp` > `cd mydockerapp`.
3. Generate a basic Maven Java project: `mvn archetype:generate -DgroupId=com.example -DartifactId=mydockerapp -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false`.
4. Navigate to the project folder: `cd mydockerapp`.
5. Edit the main Java class for testing (using Notepad or any editor): Open `src/main/java/com/example/App.java` and modify to:
   ```
   package com.example;
   public class App {
       public static void main(String[] args) {
           System.out.println("Hello from Dockerized Jenkins Pipeline!");
       }
   }
   ```

## Step 2: Create Dockerfile
1. In the project root (mydockerapp), create a file named `Dockerfile` using a text editor:
   ```
   FROM openjdk:11-jre-slim
   WORKDIR /app
   COPY target/mydockerapp-1.0-SNAPSHOT.jar /app/app.jar
   CMD ["java", "-jar", "app.jar"]
   ```
   This Dockerfile uses a slim Java runtime, copies the built JAR, and runs it.

## Step 3: Create Jenkinsfile in Project
1. In the project root, create a file named `Jenkinsfile`:
   ```
   pipeline {
       agent any
       tools {
        jdk 'OpenJDK8' <tools name inside jenkins>
        maven 'Maven3' <tool name inside jenkins>
       }
       environment {
        APP_NAME      = '<your-docker-app-name>'
        DOCKER_IMAGE  = '<your-dockerhub-username>/<your-docker-app-name>'  // Replace with your Docker Hub username
        IMAGE_TAG     = "${env.BUILD_NUMBER}"                 // Optional: Use build number as tag or give custom as abc123
       }
       stages {
           stage('SCM') {
               steps {
                   git branch: 'main', url: 'https://github.com/avv10102004/docker-spring-boot-java-web-service-example.git'  // Replace with your repo URL
               }
           }
           stage('Maven Build') {
               steps {
                   bat 'mvn clean install'
               }
           }
           stage('Build Docker Image') {
               steps {
                   bat 'docker build -t %DOCKER_IMAGE% .'
               }
           }
           stage('Docker Build & Push') {
               steps {
                   <!-- withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                       bat 'docker login -u %DOCKER_USERNAME% -p %DOCKER_PASSWORD%'
                       bat 'docker push %DOCKER_IMAGE%'
                   } -->
                   <!-- You can try this one or below one with Generate Pipeline script here no need to login it uses jenkins credentials and automatically logs in -->
                   script {
                    withDockerRegistry(credentialsId: '<your-docker-hub-credentials>') {
                        bat 'docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} .'
                        bat "docker push ${DOCKER_IMAGE}:${IMAGE_TAG}"
                    }
                   }
               }
           }
       }
       post {
           success {
               echo "Pipeline executed successfully! Image pushed to Docker Hub."
               echo "Docker image pushed: ${DOCKER_IMAGE}:${IMAGE_TAG}"
           }
           failure {
               echo "Build or push failed. Check console output."
           }
       }
   }
   ```
   This pipeline checks out code, builds the JAR with Maven, builds the Docker image, and pushes it to Docker Hub using credentials.

## Step 4: Initialize Git and Push to GitHub
1. Initialize Git: `git init`.
2. Add all files: `git add .`.
3. Commit: `git commit -m "Initial commit - mydockerapp"`.
4. Create a new repository on GitHub: Log in to github.com > "+" > New repository > Name: "mydockerapp" > Create.
5. Add remote: `git remote add origin https://github.com/<github-username>/mydockerapp.git`.
6. Set branch: `git branch -M main`.
7. Push: `git push -u origin main` (use PAT if prompted; generate at GitHub Settings > Developer settings > Personal access tokens).

## Step 5: Configure Maven and Add Credentials in Jenkins
1. Install Maven on Windows and add its `bin` directory to the system PATH (e.g., C:\maven\bin).
2. Restart Jenkins.
3. In Jenkins: Dashboard > Manage Jenkins > Global Tool Configuration > Maven > Add Maven > Name: "Maven3" > Install automatically > Save.
4. Add Docker Hub credentials: Manage Jenkins > Credentials > System > Global credentials > Add Credentials > Kind: Username with password > Username: your Docker Hub username > Password: your Docker Hub password or access token > ID: "docker-hub-creds" > OK.

## Step 6: Create Pipeline Job
1. In Jenkins: Dashboard > New Item > Pipeline > Name: "DockerPushPipeline" > OK.
2. Pipeline section > Definition: Pipeline script from SCM.
3. SCM: Git > Repository URL: https://github.com/<github-username>/mydockerapp.git > Credentials: -none- (for public repo) > Branches to build: */main > Script Path: Jenkinsfile (default) > Save.

## Step 7: Run Pipeline
1. On the job page: Click "Build Now".
2. Monitor the build: Click the build number > Console Output.
3. Verify: Check for success messages like "Pipeline executed successfully!". Log in to Docker Hub > Repositories > See "mydockerapp:latest" pushed.
4. Test locally (optional): `docker pull <dockerhub-username>/mydockerapp:latest` > `docker run <dockerhub-username>/mydockerapp:latest` (should print "Hello from Dockerized Jenkins Pipeline!").

## Where to Put Files/Credentials
- Jenkinsfile, Dockerfile, App.java, pom.xml: In the GitHub repo root (mydockerapp).
- Credentials: Docker Hub creds stored in Jenkins Credentials store (ID: "docker-hub-creds"); no extra for public GitHub repo.

## Conclusion
This assignment demonstrates creating a Jenkins pipeline that automates building a Java application with Maven, creating a Docker image, and pushing it to Docker Hub. All files (Dockerfile, Jenkinsfile, app) are stored in a GitHub repository, enabling version control and CI/CD for containerized applications.