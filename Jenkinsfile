pipeline {
    agent any

    environment {
        AZURE_ACR_NAME = 'labacrdevops2025'
        ACR_LOGIN_SERVER = "${AZURE_ACR_NAME}.azurecr.io"

        # Original names (from peer)
        ORIGINAL_IMAGE_FRONTEND = "${ACR_LOGIN_SERVER}/opendevops-nyctaxiweb-optional-frontend:v1.0.0"
        ORIGINAL_IMAGE_BACKEND  = "${ACR_LOGIN_SERVER}/opendevops-nyctaxiweb-optional-backend:v1.0.0"

        # Local retagged names (optional)
        LOCAL_IMAGE_FRONTEND = "my-custom-frontend:latest"
        LOCAL_IMAGE_BACKEND  = "my-custom-backend:latest"

        # Container names (change freely)
        CONTAINER_FRONTEND = "frontend-renamed"
        CONTAINER_BACKEND  = "backend-renamed"
    }

    stages {
        stage('Login to ACR') {
            steps {
                script {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'jenkins-acr-scope-map-cred',
                            usernameVariable: 'ACR_USER',
                            passwordVariable: 'ACR_PASS'
                        )
                    ]) {
                        sh """
                            echo "Logging in to ACR..."
                            docker login ${ACR_LOGIN_SERVER} -u \$ACR_USER -p \$ACR_PASS
                        """
                    }
                }
            }
        }

        stage('Pull & Retag Images') {
            steps {
                sh """
                    echo "Pulling images from ACR..."
                    docker pull ${ORIGINAL_IMAGE_FRONTEND}
                    docker pull ${ORIGINAL_IMAGE_BACKEND}

                    echo "Retagging images locally..."
                    docker tag ${ORIGINAL_IMAGE_FRONTEND} ${LOCAL_IMAGE_FRONTEND}
                    docker tag ${ORIGINAL_IMAGE_BACKEND} ${LOCAL_IMAGE_BACKEND}
                """
            }
        }

        stage('Stop & Remove Old Containers') {
            steps {
                sh """
                    docker stop ${CONTAINER_FRONTEND} || echo "No existing frontend"
                    docker rm ${CONTAINER_FRONTEND} || echo "No frontend to remove"

                    docker stop ${CONTAINER_BACKEND} || echo "No existing backend"
                    docker rm ${CONTAINER_BACKEND} || echo "No backend to remove"
                """
            }
        }

        stage('Deploy New Containers') {
            steps {
                sh """
                    echo "Running containers..."
                    docker run -d --name ${CONTAINER_FRONTEND} -p 7000:80 ${LOCAL_IMAGE_FRONTEND}
                    docker run -d --name ${CONTAINER_BACKEND} -p 2000:3000 ${LOCAL_IMAGE_BACKEND}

                    echo "Currently running containers:"
                    docker ps
                """
            }
        }
    }
}