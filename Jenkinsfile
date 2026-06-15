pipeline {
    agent any

    environment {
        DOCKER_USERNAME = 'mscforever'
        IMAGE_NAME = "${DOCKER_USERNAME}/docjen-nginx"
        TAG = "build-${BUILD_NUMBER}"
        UID = '976'
        XDG_RUNTIME_DIR = "/run/user/${UID}"
        PATH = "/usr/local/bin:/usr/bin:/bin"
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
                        # Проверяем окружение
                        echo "=== Environment ==="
                        echo "UID: ${UID}"
                        echo "XDG_RUNTIME_DIR: ${XDG_RUNTIME_DIR}"
                        echo "USER: \$(whoami)"
                        echo "GROUPS: \$(id)"
                        
                        # Убеждаемся что runtime директория существует
                        if [ ! -d "${XDG_RUNTIME_DIR}" ]; then
                            mkdir -p ${XDG_RUNTIME_DIR}
                            chmod 700 ${XDG_RUNTIME_DIR}
                        fi
                        
                        # Переходим в директорию с кодом
                        cd DocJen
                        
                        # Инициализируем podman
                        podman system migrate
                        
                        # Собираем образ
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
                            cd DocJen
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
            echo "🐳 https://hub.docker.com/r/${IMAGE_NAME}"
        }
        failure {
            echo "❌ Ошибка в пайплайне. Проверьте логи выше."
        }
    }
}
