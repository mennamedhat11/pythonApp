pipeline {
    agent any

    environment {
        DOCKERHUB_USER  = 'mennamedhat123'
        IMAGE_NAME      = "${DOCKERHUB_USER}/python-app"
        DOCKERHUB_CREDS = 'Docker'
    }

    stages {
        stage('Clone') {
            steps {
                checkout scm
            }
        }

        stage('Set Environment') {
            steps {
                script {
                    def branch = env.BRANCH_NAME ?: env.GIT_BRANCH?.replaceAll('origin/', '')

                    if (branch == 'prod') {
                        env.APP_ENV        = 'prod'
                        env.HOST_PORT      = '5002'
                    } else if (branch == 'stg') {
                        env.APP_ENV        = 'stg'
                        env.HOST_PORT      = '5001'
                    } else {
                        env.APP_ENV        = 'dev'
                        env.HOST_PORT      = '5000'
                    }

                    env.CONTAINER_NAME = "python-app-${env.APP_ENV}"
                    env.IMAGE_TAG      = "${env.APP_ENV}-${env.BUILD_NUMBER}"
                    env.LATEST_TAG     = "${env.APP_ENV}-latest"

                    echo "Branch: ${branch} | Env: ${env.APP_ENV} | Container: ${env.CONTAINER_NAME} | Port: ${env.HOST_PORT}"
                }
            }
        }

        stage('Build Image') {
            steps {
                sh """
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} -t ${IMAGE_NAME}:${LATEST_TAG} .
                """
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKERHUB_CREDS}",
                    usernameVariable: 'DH_USER',
                    passwordVariable: 'DH_PASS')]) {
                    sh """
                        echo "\$DH_PASS" | docker login -u "\$DH_USER" --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${IMAGE_NAME}:${LATEST_TAG}
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
                        -e APP_ENV=${APP_ENV} \
                        -p ${HOST_PORT}:5000 \
                        ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Smoke Test') {
            steps {
                sh """
                    sleep 5
                    docker exec ${CONTAINER_NAME} python -c "import urllib.request; print(urllib.request.urlopen('http://localhost:5000/health').read())"
                """
            }
        }
    }

    post {
        success { echo "Python app (${env.APP_ENV}) pipeline finished successfully" }
        failure { echo "Python app (${env.APP_ENV}) pipeline failed" }
        always  { sh 'docker image prune -f || true' }
    }
}
