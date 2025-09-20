pipeline {
    agent any

    tools {
        maven 'maven'   // défini dans Jenkins
        jdk 'jdk17'     // défini dans Jenkins
    }

    environment {
        DOCKER_HUB_USER = "bintabdallah"
        IMAGE_NAME = "springwithjava"
        RENDER_SERVICE_ID = "srv-d37a4sogjchc73c3rhr0"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/BintAbdalla/springwithjava.git',
                    credentialsId: 'github-credit'
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
                    docker.build("${DOCKER_HUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER}")
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                script {
                    def dockerImage = docker.build("${DOCKER_HUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER}")
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials') {
                        dockerImage.push()
                        dockerImage.push("latest")
                    }
                }
            }
        }

        stage('Deploy Locally (Docker Run)') {
            steps {
                script {
                    echo '🚀 Déploiement Spring local en cours...'

                    sh """
                        if [ \$(docker ps -aq -f name=${IMAGE_NAME}) ]; then
                            docker stop ${IMAGE_NAME} || true
                            docker rm ${IMAGE_NAME} || true
                        fi
                    """

                    def deployPort = null
                    def ports = [8080,8081,8082,8083,8084]

                    for (port in ports) {
                        echo "🔍 Test du port ${port}..."
                        def result = sh(script: "docker run -d -p ${port}:8080 --name ${IMAGE_NAME} ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest", returnStatus: true)
                        if (result == 0) {
                            deployPort = port
                            echo "✅ Container démarré sur le port ${port}"
                            break
                        } else {
                            echo "⚠️ Port ${port} occupé, nettoyage..."
                            sh """
                                docker stop ${IMAGE_NAME} || true
                                docker rm ${IMAGE_NAME} || true
                            """
                        }
                    }

                    if (deployPort == null) {
                        error "❌ Aucun port disponible trouvé dans la plage 8080-8084"
                    }

                    sh "sleep 5"
                    echo "🌐 Application Spring locale accessible sur : http://localhost:${deployPort}"
                }
            }
        }

        stage('Deploy to Render') {
            steps {
                withCredentials([string(credentialsId: 'render-api-key', variable: 'RENDER_API_KEY')]) {
                    httpRequest(
                        httpMode: 'POST',
                        url: "https://api.render.com/v1/services/${RENDER_SERVICE_ID}/deploys",
                        customHeaders: [[name: 'Authorization', value: "Bearer ${RENDER_API_KEY}"]],
                        contentType: 'APPLICATION_JSON',
                        validResponseCodes: '200:299'
                    )
                    echo '✅ Déploiement Render déclenché'
                }
            }
        }
    }

    post {
        always {
            echo "🧹 Nettoyage de l'espace de travail..."
            cleanWs()
        }
        failure {
            echo '❌ Pipeline échoué !'
            script {
                sh """
                    if [ \$(docker ps -aq -f name=${IMAGE_NAME}) ]; then
                        docker stop ${IMAGE_NAME} || true
                        docker rm ${IMAGE_NAME} || true
                    fi
                """
            }
        }
    }
}
