pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'c1269293-12a7-4ae4-a1fe-b048736d5658'
        SLACK_CHANNEL = '#pipeline-notifications'
        SLACK_TOKEN_CREDENTIAL_ID = 'slack-token'
        GIT_COMMIT = sh(script: 'git rev-parse --verify HEAD', returnStdout: true).trim()
        CHANGED_SERVICES = ""
    }

    stages {
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
                    def authorName = scmVars.GIT_COMMITTER_NAME
                    currentBuild.description = "Committed by: ${authorName}"
                    echo "Committer Name: ${authorName}"
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

                    env.CHANGED_SERVICES = changedServices.join(',')
                    echo "Changed Services: ${env.CHANGED_SERVICES}"
                }
            }
        }

        stage('Build Docker Images') {
            when {
                expression { return env.CHANGED_SERVICES }
            }
            steps {
                script {
                    def shortCommitHash = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
                    def services = env.CHANGED_SERVICES.split(',')

                    services.each { service ->
                        dockerBuild("imzainazm/${service}", shortCommitHash, "./${service}")
                    }
                }
            }
        }

        stage('Push Docker Images to Docker Hub') {
            when {
                expression { return env.CHANGED_SERVICES }
            }
            steps {
                script {
                    def shortCommitHash = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
                    def services = env.CHANGED_SERVICES.split(',')

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
                    def authorName = "Unknown"
                    if (currentBuild.changeSets.size() > 0 && currentBuild.changeSets.first().items.size() > 0) {
                        authorName = currentBuild.changeSets.first().items.first().author.fullName
                    }
                    echo "Committer Name: ${authorName}"
                }
            }
        }        

    post {
        success {
            script {
                if (env.CHANGED_SERVICES) {
                    sendSlackNotification(true)
                } else {
                    sendSlackNotificationNoChange(true)
                }
                cleanupImages()
            }
        }
        failure {
            script {
                if (env.CHANGED_SERVICES) {
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
    if (env.CHANGED_SERVICES) {
        def pipelineStatus = isSuccess ? "Succeeded" : "Failed"
        def triggerUser = currentBuild.rawBuild.getCause(hudson.model.Cause$UserIdCause)?.userName ?: 'Anonymous'
        def changedServices = env.CHANGED_SERVICES.replace(',', ', ')
        def environmentName = env.JOB_NAME.split('/')[0] ?: 'Unknown'

        slackSend(
            botUser: true,
            channel: SLACK_CHANNEL,
            color: isSuccess ? '#00ff00' : '#ff0000',
            message: "Pipeline ${pipelineStatus}\nCommitted by: ${currentBuild.description}\nChanged Services: ${changedServices}\nEnvironment: ${environmentName}",
            tokenCredentialId: SLACK_TOKEN_CREDENTIAL_ID
        )
    } else {
        echo "No services changed, skipping Slack notification."
    }
}

def sendSlackNotificationNoChange(isSuccess) {
    def pipelineStatus = isSuccess ? "Succeeded" : "Failed"
    def triggerUser = currentBuild.rawBuild.getCause(hudson.model.Cause$UserIdCause)?.userName ?: 'Anonymous'
    def environmentName = env.JOB_NAME.split('/')[0] ?: 'Unknown'

    slackSend(
        botUser: true,
        channel: SLACK_CHANNEL,
        color: isSuccess ? '#00ff00' : '#ff0000',
        message: "Pipeline ${pipelineStatus}\nCommitted by: ${currentBuild.description}\nNo services changed\nEnvironment: ${environmentName}",
        tokenCredentialId: SLACK_TOKEN_CREDENTIAL_ID
    )
}

def cleanupImages() {
    sh 'docker image prune -af'
}
