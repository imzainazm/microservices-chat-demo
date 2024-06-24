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
                    currentBuild.description = "Triggered by: ${scmVars.GIT_AUTHOR_NAME}"
                }
            }
        }
        stage('Get Committer Name') {
            steps {
                script {
                  def authorName = sh script: "git show -s --pretty=\"%an\" ${env.GIT_COMMIT}", returnStdout: true
                  echo "Committer Name: ${authorName.trim()}"
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
                env.GIT_COMMIT_MSG = sh (script: 'git log -1 --pretty=%B ${GIT_COMMIT}', returnStdout: true).trim()
                env.GIT_AUTHOR = sh (script: 'git log -1 --pretty=%cn ${GIT_COMMIT}', returnStdout: true).trim()
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

def lastCommiterEmail = sh(returnStdout: true, script: 'git log --format="%ae" | head -1').trim()

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
        message: "Pipeline ${pipelineStatus}\nTriggered by: ${author.trim()}\nChanged Services: ${env.changedServices}\nEnvironment: ${environmentName}",
        tokenCredentialId: SLACK_TOKEN_CREDENTIAL_ID
    )
}

def cleanupImages() {
    sh 'docker system prune -af'
}
