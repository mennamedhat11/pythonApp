pipeline {
    agent any

    environment {
        // غيّري القيم دي حسب بياناتك
        DOCKERHUB_USER = 'mennamedhat123'
        IMAGE_NAME     = "${DOCKERHUB_USER}/node-app"
        IMAGE_TAG      = "${env.BUILD_NUMBER}"
        CONTAINER_NAME = 'node-app'
        HOST_PORT      = '3000'
        DOCKERHUB_CREDS = 'dockerhub-creds'
    }

    stages {
        stage('Clone') {
            steps {
                checkout scm
            }
        }

        stage('Build Image') {
            steps {
                sh """
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} -t ${IMAGE_NAME}:latest .
                """
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${Docker}",
                    usernameVariable: 'DH_USER',
                    passwordVariable: 'DH_PASS')]) {
                    sh """
                        echo "\$DH_PASS" | docker login -u "\$DH_USER" --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${IMAGE_NAME}:latest
                        docker logout
                    """
                }
            }
        }

        stage('Deploy') {
            steps {
                sh """
                    docker rm -f ${CONTAINER_NAME} || true
                    docker run -d --name ${CONTAINER_NAME} \
                        --restart unless-stopped \
                        -p ${HOST_PORT}:3000 \
                        ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Smoke Test') {
            steps {
                sh """
                    sleep 5
                    curl -f http://localhost:${HOST_PORT}/health
                """
            }
        }
    }

    post {
        success { echo 'Node app pipeline finished successfully' }
        failure { echo 'Node app pipeline failed' }
        always  { sh 'docker image prune -f || true' }
    }
}
