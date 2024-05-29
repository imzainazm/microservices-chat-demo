pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                // Checkout your source code from your GitHub repository
                git url: 'https://github.com/imzainazm/microservices-chat-demo.git', branch: 'feature/jenkins-cicd'
            }
        }

        stage('Build Docker Image') {
            steps {
                // Build Docker image with sudo privileges
                script {
                    sh 'docker build -t imzainazm/api-gateway:latest ./api-gateway'
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                // Push Docker image to Docker Hub
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'c1269293-12a7-4ae4-a1fe-b048736d5658') {
                        docker.image('imzainazm/api-gateway:latest').push()
                    }
                }
            }
        }
    }
}
