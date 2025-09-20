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

                    // Trouver un port disponible
                    def deployPort = "8080"
                    def ports = [8080, 8081, 8082, 8083, 8084]

                    for (port in ports) {
                        def portUsed = sh(
                            script: "lsof -i:${port} > /dev/null 2>&1",
                            returnStatus: true
                        ) == 0

                        if (!portUsed) {
                            echo "✅ Port ${port} disponible"
                            deployPort = port.toString()
                            break
                        } else {
                            echo "⚠️ Port ${port} occupé"
                        }
                    }

                    if (deployPort == "8080" && sh(script: "lsof -i:8080 > /dev/null 2>&1", returnStatus: true) == 0) {
                        error "❌ Aucun port disponible trouvé dans la plage 8080-8084"
                    }
                    echo "📍 Déploiement sur le port ${deployPort}"

                    // Lancer le container Spring Boot
                    sh """
                        echo "Lancement du container sur le port ${deployPort}..."
                        docker run -d -p ${deployPort}:8080 --name ${IMAGE_NAME} ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest
                        echo "Attente du démarrage du container..."
                        sleep 5
                        if docker ps | grep ${IMAGE_NAME} > /dev/null; then
                            echo "✅ Container Spring Boot démarré avec succès sur le port ${deployPort}"
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