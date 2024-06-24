pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'c1269293-12a7-4ae4-a1fe-b048736d5658'
        SLACK_CHANNEL = '#pipeline-notifications'
        SLACK_TOKEN_CREDENTIAL_ID = 'slack-token'
        CHANGED_SERVICES = ''
        COMMITTER_NAME = ''
    }

    stages {
        stage('Fetch Committer Info') {
            steps {
                script {
                    def commitInfo = sh(script: 'git log -1 --pretty=%an,%ae', returnStdout: true).trim().split(',')
                    COMMITTER_NAME = commitInfo[0]?.trim() ?: 'Unknown'
                    echo "Committer Name: ${COMMITTER_NAME}"
                }
            }
        }

        stage('Checkout') {
            steps {
                script {
                    def scmVars = checkout scm: [
                        $class: 'GitSCM',
                        branches: [[name: '*/develop']],
                        doGenerateSubmoduleConfigurations: false,
                        extensions: [],
                        userRemoteConfigs: [[url: 'https://github.com/imzainazm/microservices-chat-demo.git']]
                    ]
                    currentBuild.description = "Committed by: ${COMMITTER_NAME}"
                    echo "Committer Name: ${COMMITTER_NAME}"
                }
            }
        }

        stage('Determine Changes') {
            steps {
                script {
                    def changedFiles = sh(
                        script: 'git diff --name-only HEAD~1 HEAD',
                        returnStdout: true
                    ).trim().split('\n')

                    def changedServices = []

                    if (changedFiles.any { it.startsWith('api-gateway/') }) {
                        changedServices.add('api-gateway')
                    }
                    if (changedFiles.any { it.startsWith('users-service/') }) {
                        changedServices.add('users-service')
                    }
                    if (changedFiles.any { it.startsWith('chat-service/') }) {
                        changedServices.add('chat-service')
                    }
                    if (changedFiles.any { it.startsWith('chat-app/') }) {
                        changedServices.add('chat-app')
                    }

                    CHANGED_SERVICES = changedServices.join(',')
                    echo "Changed Services: ${CHANGED_SERVICES}"
                }
            }
        }

        stage('Build Docker Images') {
            when {
                expression { return CHANGED_SERVICES != '' }
            }
            steps {
                script {
                    def shortCommitHash = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
                    def services = CHANGED_SERVICES.split(',')

                    services.each { service ->
                        dockerBuild("imzainazm/${service}", shortCommitHash, "./${service}")
                    }
                }
            }
        }

        stage('Push Docker Images to Docker Hub') {
            when {
                expression { return CHANGED_SERVICES != '' }
            }
            steps {
                script {
                    def shortCommitHash = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
                    def services = CHANGED_SERVICES.split(',')

                    parallel services.collectEntries { service ->
                        ["Push ${service.capitalize()}": {
                            dockerPush("imzainazm/${service}", shortCommitHash)
                        }]
                    }
                }
            }
        }

        stage('Get Committer Name') {
            steps {
                script {
                    def commitInfo = sh(script: 'git log -1 --pretty=%an,%ae', returnStdout: true).trim().split(',')
                    COMMITTER_NAME = commitInfo[0]?.trim() ?: 'Unknown'
                    echo "Committer Name: ${COMMITTER_NAME}"
                }
            }
        }
    }

    post {
        success {
            script {
                if (CHANGED_SERVICES) {
                    sendSlackNotification(true)
                } else {
                    sendSlackNotificationNoChange(true)
                }
                cleanupImages()
            }
        }
        failure {
            script {
                if (CHANGED_SERVICES) {
                    sendSlackNotification(false)
                } else {
                    sendSlackNotificationNoChange(false)
                }
                cleanupImages()
            }
        }
    }
}

def dockerBuild(imageName, tag, dockerfilePath) {
    sh "docker build -t ${imageName}:latest -t ${imageName}:${tag} ${dockerfilePath}"
}

def dockerPush(imageName, tag) {
    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
        docker.image("${imageName}:latest").push()
        docker.image("${imageName}:${tag}").push()
    }
}

def sendSlackNotification(isSuccess) {
    def pipelineStatus = isSuccess ? "Succeeded" : "Failed"
    def changedServices = CHANGED_SERVICES.replaceAll(',', ', ')
    def environmentName = env.JOB_NAME.split('/')[0] ?: 'Unknown'

    slackSend(
        botUser: true,
        channel: SLACK_CHANNEL,
        color: isSuccess ? '#00ff00' : '#ff0000',
        message: "Pipeline ${pipelineStatus}\nCommitted by: ${COMMITTER_NAME}\nChanged Services: ${changedServices}\nEnvironment: ${environmentName}",
        tokenCredentialId: SLACK_TOKEN_CREDENTIAL_ID
    )
}

def sendSlackNotificationNoChange(isSuccess) {
    def pipelineStatus = isSuccess ? "Succeeded" : "Failed"
    def environmentName = env.JOB_NAME.split('/')[0] ?: 'Unknown'

    slackSend(
        botUser: true,
        channel: SLACK_CHANNEL,
        color: isSuccess ? '#00ff00' : '#ff0000',
        message: "Pipeline ${pipelineStatus}\nCommitted by: ${COMMITTER_NAME}\nNo services changed\nEnvironment: ${environmentName}",
        tokenCredentialId: SLACK_TOKEN_CREDENTIAL_ID
    )
}

def cleanupImages() {
    sh 'docker image prune -af'
}
