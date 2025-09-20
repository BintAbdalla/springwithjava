pipeline {
    agent any

    tools {
        maven 'maven'   // défini dans Jenkins
        jdk 'jdk17'     // défini dans Jenkins
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

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def dockerImage = docker.build("${DOCKER_HUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER}")
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

        stage('Deploy (Docker Run)') {
            steps {
                script {
                    echo '🚀 Déploiement Spring en cours...'

                    // Arrêter et supprimer un container existant
                    sh """
                        if [ \$(docker ps -aq -f name=${IMAGE_NAME}) ]; then
                            echo "Arrêt du container existant..."
                            docker stop ${IMAGE_NAME} || true
                            docker rm ${IMAGE_NAME} || true
                            echo "Container existant supprimé"
                        fi
                    """

                    // Solution simple : essayer les ports un par un avec Docker directement
                    def deployPort = null
                    def ports = [8080, 8081, 8082, 8083, 8084]

                    for (port in ports) {
                        echo "🔍 Test du port ${port}..."

                        def result = sh(
                            script: """
                                docker run -d -p ${port}:8080 --name ${IMAGE_NAME} ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest
                            """,
                            returnStatus: true
                        )

                        if (result == 0) {
                            deployPort = port
                            echo "✅ Container démarré avec succès sur le port ${port}"
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

                    // Vérifier que le container fonctionne
                    sh """
                        echo "Attente du démarrage du container..."
                        sleep 5
                        if docker ps | grep ${IMAGE_NAME} > /dev/null; then
                            echo "✅ Container Spring Boot vérifié et fonctionnel"
                        else
                            echo "❌ Erreur lors du démarrage du container"
                            docker logs ${IMAGE_NAME} || true
                            exit 1
                        fi
                    """

                    echo "🌐 Application Spring accessible sur : http://localhost:${deployPort}"
                }
            }
        }
    }

    post {
        always {
            echo "🧹 Nettoyage de l'espace de travail..."
            cleanWs()
        }
        success {
            echo '✅ Pipeline exécuté avec succès !'
            echo '🎉 L\'application a été déployée correctement'
        }
        failure {
            echo '❌ Pipeline échoué !'
            script {
                echo "🧹 Nettoyage des containers en cas d'échec..."
                sh """
                    if [ \$(docker ps -aq -f name=${IMAGE_NAME}) ]; then
                        echo "Suppression du container défaillant..."
                        docker stop ${IMAGE_NAME} || true
                        docker rm ${IMAGE_NAME} || true
                        echo "Nettoyage terminé"
                    fi
                """
            }
        }
    }
}