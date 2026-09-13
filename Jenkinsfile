pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        AWS_ACCOUNT_ID = '938521951639'
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"

        FRONTEND_IMAGE = "${ECR_REGISTRY}/streamingapp-frontend"
        AUTH_IMAGE = "${ECR_REGISTRY}/streamingapp-auth"
        STREAMING_IMAGE = "${ECR_REGISTRY}/streamingapp-streaming"
        ADMIN_IMAGE = "${ECR_REGISTRY}/streamingapp-admin"
        CHAT_IMAGE = "${ECR_REGISTRY}/streamingapp-chat"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Check Tools') {
            steps {
                sh '''
                    docker --version
                    aws --version
                    git --version
                '''
            }
        }

        stage('Login to ECR') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                     credentialsId: 'aws-ecr-jenkins']
                ]) {
                    sh '''
                        aws ecr get-login-password --region $AWS_REGION |
                        docker login --username AWS --password-stdin $ECR_REGISTRY
                    '''
                }
            }
        }

        stage('Build Images') {
            steps {
                sh '''
                    docker build \
                      -t $FRONTEND_IMAGE:$BUILD_NUMBER \
                      ./frontend

                    docker build \
                      -t $AUTH_IMAGE:$BUILD_NUMBER \
                      ./backend/authService

                    docker build \
                      -t $STREAMING_IMAGE:$BUILD_NUMBER \
                      -f ./backend/streamingService/Dockerfile \
                      ./backend

                    docker build \
                      -t $ADMIN_IMAGE:$BUILD_NUMBER \
                      -f ./backend/adminService/Dockerfile \
                      ./backend

                    docker build \
                      -t $CHAT_IMAGE:$BUILD_NUMBER \
                      -f ./backend/chatService/Dockerfile \
                      ./backend
                '''
            }
        }

        stage('Push Images') {
            steps {
                sh '''
                    docker push $FRONTEND_IMAGE:$BUILD_NUMBER
                    docker push $AUTH_IMAGE:$BUILD_NUMBER
                    docker push $STREAMING_IMAGE:$BUILD_NUMBER
                    docker push $ADMIN_IMAGE:$BUILD_NUMBER
                    docker push $CHAT_IMAGE:$BUILD_NUMBER
                '''
            }
        }
    }

    post {
        success {
            echo 'All Docker images successfully pushed to ECR.'
        }

        failure {
            echo 'Pipeline failed. Check the Jenkins console output.'
        }
    }
}
