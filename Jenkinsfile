pipeline {
    agent any
    
    environment {
        DOCKER_CREDENTIALS_ID = 'c1269293-12a7-4ae4-a1fe-b048736d5658'
        SLACK_CHANNEL = '#pipeline-notifications'
        SLACK_TOKEN_CREDENTIAL_ID = 'slack-token'
        GIT_COMMIT = sh script: 'git rev-parse --verify HEAD', returnStdout: true
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
                    def authorName = currentBuild.changeSets.first()?.items?.first()?.author?.fullName
                    currentBuild.description = "Triggered by: ${authorName}"
                    echo "Committer Name: ${authorName}"
                }
            }
        }
        stage('Get Committer Name') {
            steps {
                script {
                    def author = sh script: "git show -s --pretty=\"%an\" ${GIT_COMMIT}", returnStdout: true
                    echo "Committer Name: ${author?.trim() ?: 'Author Name Not Available'}"
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
                    
                    env.CHANGED_SERVICES = []
                    if (changedFiles.any { it.startsWith('api-gateway/') }) {
                        env.CHANGED_SERVICES += 'api-gateway'
                        echo "Changed Services: ${env.CHANGED_SERVICES}"
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

        stage('Build Docker Images') {
            when {
                expression { env.CHANGED_SERVICES }
            }
            steps {
                script {
                    def shortCommitHash = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
                    
                    if (env.CHANGED_SERVICES.contains('api-gateway')) {
                        dockerBuild('imzainazm/api-gateway', shortCommitHash, './api-gateway')
                    }
                    if (env.CHANGED_SERVICES.contains('users-service')) {
                        dockerBuild('imzainazm/users-service', shortCommitHash, './users-service')
                    }
                    if (env.CHANGED_SERVICES.contains('chat-service')) {
                        dockerBuild('imzainazm/chat-service', shortCommitHash, './chat-service')
                    }
                    if (env.CHANGED_SERVICES.contains('chat-app')) {
                        dockerBuild('imzainazm/chat-app', shortCommitHash, './chat-app')
                    }
                }
            }
        }

        stage('Push Docker Images to Docker Hub') {
            when {
                expression { env.CHANGED_SERVICES }
            }
            steps {
                script {
                    def shortCommitHash = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
                    
                    parallel (
                        "Push API Gateway": {
                            dockerPush('imzainazm/api-gateway', shortCommitHash)
                        },
                        "Push Users Service": {
                            dockerPush('imzainazm/users-service', shortCommitHash)
                        },
                        "Push Chat Service": {
                            dockerPush('imzainazm/chat-service', shortCommitHash)
                        },
                        "Push Chat App": {
                            dockerPush('imzainazm/chat-app', shortCommitHash)
                        }
                    )
                }
            }
        }
    }

    stage('get_commit_details') {
      steps {
        script {
          def authorName1 = currentBuild.changeSets.first()?.items?.first()?.author?.fullName
          echo "Committer Name (redundant): ${authorName1}"
        }
      }
    }

    post {
        success {
            script {
                sendSlackNotification(true)
                cleanupImages()
            }
        }
        failure {
            script {
                sendSlackNotification(false)
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
    def triggerUser = currentBuild.rawBuild.getCause(Cause.UserIdCause).userName ?: 'Anonymous'
    def changedServices = env.CHANGED_SERVICES.join(', ')
    def environmentName = env.JOB_NAME.split('/')[0] ?: 'Unknown'
    
    slackSend(
        botUser: true,
        channel: SLACK_CHANNEL,
        color: isSuccess ? '#00ff00' : '#ff0000',
        message: "Pipeline ${pipelineStatus}\n${currentBuild.description}\nChanged Services: ${env.changedServices}\nEnvironment: ${environmentName}\n",
        tokenCredentialId: SLACK_TOKEN_CREDENTIAL_ID
    )
}

def cleanupImages() {
    sh 'docker system prune -af'
}
