pipeline {
    agent any
    environment {
        AWS_REGION      = 'ap-south-1'
        ECR_REGISTRY    = '431445330886.dkr.ecr.ap-south-1.amazonaws.com'
        IMAGE_FRONTEND  = "${ECR_REGISTRY}/streamingapp-frontend"
        IMAGE_AUTH      = "${ECR_REGISTRY}/streamingapp-auth"
        IMAGE_STREAMING = "${ECR_REGISTRY}/streamingapp-streaming"
        IMAGE_ADMIN     = "${ECR_REGISTRY}/streamingapp-admin"
        IMAGE_CHAT      = "${ECR_REGISTRY}/streamingapp-chat"
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/sanojaix-debug/StreamingApp.git'
            }
        }
        stage('Build Images') {
            steps {
                sh 'docker build -t streamingapp-frontend ./frontend'
                sh 'docker build -t streamingapp-auth ./backend/authService'
                sh 'docker build -t streamingapp-streaming -f ./backend/streamingService/Dockerfile ./backend'
                sh 'docker build -t streamingapp-admin -f ./backend/adminService/Dockerfile ./backend'
                sh 'docker build -t streamingapp-chat -f ./backend/chatService/Dockerfile ./backend'
            }
        }
        stage('Push to ECR') {
            steps {
                withCredentials([aws(
                    credentialsId: 'aws-ecr-creds',
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                )]) {
                    sh """
                        aws ecr get-login-password --region ${AWS_REGION} | \
                        docker login --username AWS --password-stdin ${ECR_REGISTRY}

                        docker tag streamingapp-frontend:latest ${IMAGE_FRONTEND}:latest
                        docker push ${IMAGE_FRONTEND}:latest

                        docker tag streamingapp-auth:latest ${IMAGE_AUTH}:latest
                        docker push ${IMAGE_AUTH}:latest

                        docker tag streamingapp-streaming:latest ${IMAGE_STREAMING}:latest
                        docker push ${IMAGE_STREAMING}:latest

                        docker tag streamingapp-admin:latest ${IMAGE_ADMIN}:latest
                        docker push ${IMAGE_ADMIN}:latest

                        docker tag streamingapp-chat:latest ${IMAGE_CHAT}:latest
                        docker push ${IMAGE_CHAT}:latest
                    """
                }
            }
        }
    }
    post {
        success { echo '✅ All images built and pushed to ECR successfully.' }
        failure { echo '❌ Build failed. Check logs above.' }
    }
}
