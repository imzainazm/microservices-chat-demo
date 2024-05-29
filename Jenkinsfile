pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                // Checkout your source code from your GitHub repository
                git url: 'https://github.com/imzainazm/microservices-chat-demo.git', branch: 'feature/jenkins-cicd'
            }
        }

        stage('Build Docker Images') {
            parallel {
                stage('Build API Gateway') {
                    steps {
                        script {
                            sh 'docker build -t imzainazm/api-gateway:latest ./api-gateway'
                        }
                    }
                }
                stage('Build Users Service') {
                    steps {
                        script {
                            sh 'docker build -t imzainazm/users-service:latest ./users-service'
                        }
                    }
                }
                stage('Build Chat Service') {
                    steps {
                        script {
                            sh 'docker build -t imzainazm/chat-service:latest ./chat-service'
                        }
                    }
                }
                stage('Build Chat App') {
                    steps {
                        script {
                            sh 'docker build -t imzainazm/chat-app:latest ./chat-app'
                        }
                    }
                }
            }
        }

        stage('Push Docker Images to Docker Hub') {
            parallel {
                stage('Push API Gateway') {
                    steps {
                        script {
                            docker.withRegistry('https://index.docker.io/v1/', 'c1269293-12a7-4ae4-a1fe-b048736d5658') {
                                docker.image('imzainazm/api-gateway:latest').push()
                            }
                        }
                    }
                }
                stage('Push Users Service') {
                    steps {
                        script {
                            docker.withRegistry('https://index.docker.io/v1/', 'c1269293-12a7-4ae4-a1fe-b048736d5658') {
                                docker.image('imzainazm/users-service:latest').push()
                            }
                        }
                    }
                }
                stage('Push Chat Service') {
                    steps {
                        script {
                            docker.withRegistry('https://index.docker.io/v1/', 'c1269293-12a7-4ae4-a1fe-b048736d5658') {
                                docker.image('imzainazm/chat-service:latest').push()
                            }
                        }
                    }
                }
                stage('Push Chat App') {
                    steps {
                        script {
                            docker.withRegistry('https://index.docker.io/v1/', 'c1269293-12a7-4ae4-a1fe-b048736d5658') {
                                docker.image('imzainazm/chat-app:latest').push()
                            }
                        }
                    }
                }
            }
        }
    }
}
