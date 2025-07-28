pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
        BACKEND_REPO = '851725361780.dkr.ecr.us-east-1.amazonaws.com/3tier-backend'
        FRONTEND_REPO = '851725361780.dkr.ecr.us-east-1.amazonaws.com/3tier-frontend'
        IMAGE_TAG = 'latest'
        APP_SERVER = 'ubuntu@3.81.224.147' // Replace with your new app server's public IP
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/wasiabdullah111/3tier-ToDoApp.git'
            }
        }

        stage('Login to AWS ECR') {
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

        stage('Push Docker Images to ECR') {
            steps {
                sh '''
                    docker push $BACKEND_REPO:$IMAGE_TAG
                    docker push $FRONTEND_REPO:$IMAGE_TAG
                '''
            }
        }

        stage('Deploy on App Server') {
            steps {
                sshagent(credentials: ['app-server-ssh']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no $APP_SERVER << 'EOF'
                        echo "Logging into ECR..."
                        aws ecr get-login-password | docker login --username AWS --password-stdin $BACKEND_REPO
                        aws ecr get-login-password | docker login --username AWS --password-stdin $FRONTEND_REPO

                        echo "Pulling latest images..."
                        docker pull $BACKEND_REPO:$IMAGE_TAG
                        docker pull $FRONTEND_REPO:$IMAGE_TAG

                        echo "Stopping and removing old containers if they exist..."
                        docker stop backend || true && docker rm backend || true
                        docker stop frontend || true && docker rm frontend || true

                        echo "Starting new containers..."
                        docker run -d --name backend -p 8080:8080 $BACKEND_REPO:$IMAGE_TAG
                        docker run -d --name frontend -p 80:3000 --link backend $FRONTEND_REPO:$IMAGE_TAG

                        echo "Deployment completed successfully."
                    EOF
                    """
                }
            }
        }
    }
}
