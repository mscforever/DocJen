pipeline {
    agent any

    environment {
        DOCKER_USERNAME = 'mscforever'
        IMAGE_NAME = "${DOCKER_USERNAME}/docjen-nginx"
        TAG = "${env.BUILD_NUMBER}-${env.BUILD_ID}"
    }

    stages {
        stage('Checkout from GitHub') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/mscforever/DocJen.git'
                echo '✅ Код успешно склонирован из репозитория DocJen'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                        podman build -t ${IMAGE_NAME}:${TAG} .
                        podman tag ${IMAGE_NAME}:${TAG} ${IMAGE_NAME}:latest
                    """
                    echo '✅ Образ собран успешно'
                }
            }
        }

        stage('Push to Docker Registry') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'docker-hub-token', variable: 'DOCKER_PASS')]) {
                        sh """
                            echo ${DOCKER_PASS} | podman login docker.io -u ${DOCKER_USERNAME} --password-stdin
                            podman push ${IMAGE_NAME}:${TAG}
                            podman push ${IMAGE_NAME}:latest
                        """
                    }
                    echo '✅ Образ отправлен в Docker Hub'
                }
            }
        }
    }

    post {
        success {
            echo "🎉 Pipeline успешно выполнен!"
            echo "📦 Образ: ${IMAGE_NAME}:${TAG}"
        }
        failure {
            echo "❌ Ошибка в пайплайне. Проверьте логи выше."
        }
    }
}
