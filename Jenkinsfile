pipeline {
    agent any

    tools {
        maven 'maven'   // Défini dans Jenkins (Global Tool Config)
        jdk 'jdk17'     // Défini aussi dans Jenkins
    }

    environment {
        DOCKER_HUB_USER = "bintabdallah"
        IMAGE_NAME = "springwithjava"
    }

stages {
    stage('Checkout') {
        steps {
            git branch: 'main',
                url: 'https://github.com/BintAbdalla/springwithjava.git',
                credentialsId: 'github-credit'
        }
    }
}
        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_HUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER}")
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials') {
                        dockerImage.push()
                        dockerImage.push("latest")
                    }
                }
            }
        }

        stage('Deploy (Docker Run)') {
            steps {
                sh '''
                docker stop demo-app || true
                docker rm demo-app || true
                docker run -d --name demo-app -p 8080:8080 ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest
                '''
            }
        }
    }
}
