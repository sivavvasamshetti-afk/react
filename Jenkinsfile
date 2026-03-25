pipeline {
    agent any

    environment {
        GIT_REPO        = "https://github.com/sivavvasamshetti-afk/react.git"
        GIT_BRANCH      = "main"

        DOCKERHUB_USER  = "sivav2516"
        IMAGE_NAME      = "react-app"
        IMAGE_TAG       = "${BUILD_NUMBER}"

        DOCKER_CREDS    = "Docker_cred"

        CONTAINER_NAME  = "react-container"
        HOST_PORT       = "8855"
        CONTAINER_PORT  = "80"
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scmGit(
                    branches: [[name: "*/${GIT_BRANCH}"]],
                    userRemoteConfigs: [[
                        credentialsId: 'shivagit',
                        url: "${GIT_REPO}"
                    ]]
                )
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} .
                """
            }
        }

        stage('DockerHub Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKER_CREDS}",
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                    echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                    """
                }
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                sh """
                docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Deploy Container') {
            steps {
                sh """
                docker stop ${CONTAINER_NAME} || true
                docker rm ${CONTAINER_NAME} || true

                docker run -d \
                -p ${HOST_PORT}:${CONTAINER_PORT} \
                --name ${CONTAINER_NAME} \
                ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }
    }

    post {
        success {
            emailext(
                subject: "SUCCESS: Build #${BUILD_NUMBER} for ${IMAGE_NAME}",
                body: "Good news!\n\nThe Jenkins pipeline completed successfully.\n\nImage: ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}\nContainer: ${CONTAINER_NAME}\nPort: ${HOST_PORT}\n\nRegards,\nJenkins",
                to: "siva.vvasamshetti@gamil.com"
            )
        }
        failure {
            emailext(
                subject: "FAILURE: Build #${BUILD_NUMBER} for ${IMAGE_NAME}",
                body: "Unfortunately, the Jenkins pipeline failed.\n\nPlease check the Jenkins console logs for details.\n\nRegards,\nJenkins",
                to: "siva.vvasamshetti@gamil.com"
            )
        }
    }
}
