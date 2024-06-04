pipeline {
    agent any
    environment {
        DOCKER_CREDENTIALS_ID = 'c1269293-12a7-4ae4-a1fe-b048736d5658'
        SLACK_CHANNEL = '#pipeline-notifications'
        SLACK_TOKEN_CREDENTIAL_ID = 'slack-token'
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    // Checkout your source code from your GitHub repository
                    def scmVars = checkout scm: [
                        $class: 'GitSCM',
                        branches: [[name: '*/develop']],
                        doGenerateSubmoduleConfigurations: false,
                        extensions: [],
                        userRemoteConfigs: [[url: 'https://github.com/imzainazm/microservices-chat-demo.git']]
                    ]
                    currentBuild.description = "Triggered by: ${scmVars.GIT_AUTHOR_NAME}"
                }
            }
        }

        stage('Determine Changes') {
            steps {
                script {
                    // Determine which services have changed
                    def changedFiles = sh(
                        script: 'git diff --name-only HEAD~1 HEAD',
                        returnStdout: true
                    ).trim().split('\n')
                    
                    env.CHANGED_SERVICES = []
                    if (changedFiles.any { it.startsWith('api-gateway/') }) {
                        env.CHANGED_SERVICES += 'api-gateway'
                    }
                    if (changedFiles.any { it.startsWith('users-service/') }) {
                        env.CHANGED_SERVICES += 'users-service'
                    }
                    if (changedFiles.any { it.startsWith('chat-service/') }) {
                        env.CHANGED_SERVICES += 'chat-service'
                    }
                    if (changedFiles.any { it.startsWith('chat-app/') }) {
                        env.CHANGED_SERVICES += 'chat-app'
                    }
                }
            }
        }

        stage('Build and Push Docker Images') {
            when {
                expression { env.CHANGED_SERVICES }
            }
            steps {
                script {
                    // List of microservice directories
                    def microservices = env.CHANGED_SERVICES
                    
                    for (microservice in microservices) {
                        // Build and push Docker image
                        sh """
                            docker build -t imzainazm/${microservice}:latest ./${microservice}
                            docker login -u your-dockerhub-username -p your-dockerhub-password
                            docker push imzainazm/${microservice}:latest
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            script {
                slackSend(
                    botUser: true,
                    channel: SLACK_CHANNEL,
                    color: '#00ff00',
                    message: "Pipeline Succeeded\nTriggered by: ${params.BUILD_CAUSE}\nUpdated Services: ${env.CHANGED_SERVICES.join(', ')}\nEnvironment: ${env.JOB_NAME}",
                    tokenCredentialId: SLACK_TOKEN_CREDENTIAL_ID
                )
            }
        }
        failure {
            script {
                slackSend(
                    botUser: true,
                    channel: SLACK_CHANNEL,
                    color: '#ff0000',
                    message: "Pipeline Failed\nTriggered by: ${params.BUILD_CAUSE}\nUpdated Services: ${env.CHANGED_SERVICES.join(', ')}\nEnvironment: ${env.JOB_NAME}",
                    tokenCredentialId: SLACK_TOKEN_CREDENTIAL_ID
                )
            }
        }
    }
}
