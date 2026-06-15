pipeline {
    agent any

    environment {
        DOCKER_USERNAME = 'mscforever'
        IMAGE_NAME = "${DOCKER_USERNAME}/docjen-nginx"
        TAG = "build-${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout from GitHub') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/mscforever/DocJen.git',
                    credentialsId: 'github-token'
                echo '✅ Код успешно склонирован'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                        export XDG_RUNTIME_DIR=/run/user/976
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
                            export XDG_RUNTIME_DIR=/run/user/976
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
