pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        AWS_ACCOUNT_ID = "354918387085"
        ECR_BACKEND_REPO = "mern-backend"
        ECR_FRONTEND_REPO = "mern-frontend"
        EC2_SSH_USER = "ubuntu"
        EC2_HOST = "44.204.213.233"
        SSH_KEY_PATH = "/var/lib/jenkins/.ssh/devops-key.pem"  // Update with the correct SSH key path
    }

    stages {
        stage('Connect to Remote Server') {
            steps {
                script {
                    sh '''
                ssh -o StrictHostKeyChecking=no -i $SSH_KEY_PATH $EC2_SSH_USER@$EC2_HOST "echo Connected to EC2"
                    '''
                }
            }
        }

        stage('Pull Latest Code') {
            steps {
                script {
                    sh '''
                    ssh -i $SSH_KEY_PATH $EC2_SSH_USER@$EC2_HOST "cd ~/MERN-DOCKER-COMPOSE && git pull origin master"
                    '''
                }
            }
        }

        stage('Authenticate with AWS ECR') {
            steps {
                script {
                    sh '''
             ssh -i $SSH_KEY_PATH $EC2_SSH_USER@$EC2_HOST "
             aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com"
            '''
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    sh '''
                    ssh -i $SSH_KEY_PATH $EC2_SSH_USER@$EC2_HOST "cd ~/MERN-DOCKER-COMPOSE && docker-compose build"
                    '''
                }
            }
        }

        stage('Push Images to AWS ECR') {
            steps {
                script {
                    sh '''
                    ssh -i $SSH_KEY_PATH $EC2_SSH_USER@$EC2_HOST "cd ~/MERN-DOCKER-COMPOSE && \
                    docker tag mern-docker-compose_backend:latest $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_BACKEND_REPO:latest && \
                    docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_BACKEND_REPO:latest && \
                    docker tag mern-docker-compose_frontend:latest $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_FRONTEND_REPO:latest && \
                    docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_FRONTEND_REPO:latest"
                    '''
                }
            }
        }

        stage('Deploy Application') {
            steps {
                script {
                    sh '''
                    ssh -i $SSH_KEY_PATH $EC2_SSH_USER@$EC2_HOST "cd ~/MERN-DOCKER-COMPOSE && docker-compose down && docker-compose up -d"
                    '''
                }
            }
        }
    }
}