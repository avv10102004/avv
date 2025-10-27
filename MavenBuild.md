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

