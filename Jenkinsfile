pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
        BACKEND_REPO = '851725361780.dkr.ecr.us-east-1.amazonaws.com/3tier-backend'
        FRONTEND_REPO = '851725361780.dkr.ecr.us-east-1.amazonaws.com/3tier-frontend'
        IMAGE_TAG = 'latest'
        APP_SERVER = 'ubuntu@3.81.224.147'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/wasiabdullah111/3tier-ToDoApp.git'
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                    aws ecr get-login-password | docker login --username AWS --password-stdin $BACKEND_REPO
                    aws ecr get-login-password | docker login --username AWS --password-stdin $FRONTEND_REPO
                '''
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                    docker build -t $BACKEND_REPO:$IMAGE_TAG ./backend
                    docker build -t $FRONTEND_REPO:$IMAGE_TAG ./frontend
                '''
            }
        }

        stage('Push Docker Images') {
            steps {
                sh '''
                    docker push $BACKEND_REPO:$IMAGE_TAG
                    docker push $FRONTEND_REPO:$IMAGE_TAG
                '''
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['app-server-ssh']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no $APP_SERVER << 'EOF'
                    aws ecr get-login-password | docker login --username AWS --password-stdin $BACKEND_REPO
                    aws ecr get-login-password | docker login --username AWS --password-stdin $FRONTEND_REPO

                    docker pull $BACKEND_REPO:$IMAGE_TAG
                    docker pull $FRONTEND_REPO:$IMAGE_TAG

                    docker stop backend || true && docker rm backend || true
                    docker stop frontend || true && docker rm frontend || true

                    docker run -d --name backend -p 8080:8080 $BACKEND_REPO:$IMAGE_TAG
                    docker run -d --name frontend -p 80:3000 --link backend $FRONTEND_REPO:$IMAGE_TAG
                    EOF
                    """
                }
            }
        }
    }
}
