pipeline {
    agent any
    parameters {
        string(name: 'AWS_KEY', defaultValue: 'AKIAWI5BFJPDI5E766H6', description: 'AWS Access Key ID')
        password(name: 'AWS_SECRET', defaultValue: '', description: 'AWS Secret Access Key')
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
                sh '''
                    mkdir -p ~/.aws
                    echo "[default]" > ~/.aws/credentials
                    echo "aws_access_key_id = $AWS_KEY" >> ~/.aws/credentials
                    echo "aws_secret_access_key = $AWS_SECRET" >> ~/.aws/credentials
                    echo "[default]" > ~/.aws/config
                    echo "region = ap-south-1" >> ~/.aws/config

                    aws ecr get-login-password --region ap-south-1 | \
                    docker login --username AWS --password-stdin 431445330886.dkr.ecr.ap-south-1.amazonaws.com

                    docker tag streamingapp-frontend:latest 431445330886.dkr.ecr.ap-south-1.amazonaws.com/streamingapp-frontend:latest
                    docker push 431445330886.dkr.ecr.ap-south-1.amazonaws.com/streamingapp-frontend:latest

                    docker tag streamingapp-auth:latest 431445330886.dkr.ecr.ap-south-1.amazonaws.com/streamingapp-auth:latest
                    docker push 431445330886.dkr.ecr.ap-south-1.amazonaws.com/streamingapp-auth:latest

                    docker tag streamingapp-streaming:latest 431445330886.dkr.ecr.ap-south-1.amazonaws.com/streamingapp-streaming:latest
                    docker push 431445330886.dkr.ecr.ap-south-1.amazonaws.com/streamingapp-streaming:latest

                    docker tag streamingapp-admin:latest 431445330886.dkr.ecr.ap-south-1.amazonaws.com/streamingapp-admin:latest
                    docker push 431445330886.dkr.ecr.ap-south-1.amazonaws.com/streamingapp-admin:latest

                    docker tag streamingapp-chat:latest 431445330886.dkr.ecr.ap-south-1.amazonaws.com/streamingapp-chat:latest
                    docker push 431445330886.dkr.ecr.ap-south-1.amazonaws.com/streamingapp-chat:latest
                '''
            }
        }
    }
    post {
        success { echo '✅ All images built and pushed to ECR successfully.' }
        failure { echo '❌ Build failed. Check logs above.' }
    }
}
