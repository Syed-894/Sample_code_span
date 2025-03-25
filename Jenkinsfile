pipeline {
    agent { 
        docker {
            image 'docker:latest'
            args '-v /var/run/docker.sock:/var/run/docker.sock -u root' // Run as root to avoid permission issues
        }
    }
    environment {
        DOCKER_REGISTRY = 'syedali161'  // Your DockerHub username
        IMAGE_NAME = "jenkins-demo-app"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        SLACK_CHANNEL = '#all-span-devops'  // Your Slack channel name
    }
    triggers {
        githubPush()
    }
    parameters {
        string(name: 'GIT_BRANCH', defaultValue: 'main', description: 'Branch to checkout')
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: "${params.GIT_BRANCH}", url: 'https://github.com/Syed-894/Sample_code_span.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    withEnv(['DOCKER_CONFIG=/tmp/.docker']) {
                        // Create the .docker directory and set permissions
                        sh '''
                            mkdir -p /tmp/.docker
                            chmod 700 /tmp/.docker
                        '''

                        // Create a .dockerignore file
                        sh 'echo ".docker" > .dockerignore'

                        // Build the Docker image
                        def dockerImage = docker.build("${env.IMAGE_NAME}:${env.IMAGE_TAG}", '.')

                        // Authenticate and push the image
                        withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASSWORD')]) {
                            sh "docker --config /tmp/.docker login -u $DOCKER_USER -p $DOCKER_PASSWORD"
                            sh "docker --config /tmp/.docker tag ${env.IMAGE_NAME}:${env.IMAGE_TAG} ${env.DOCKER_REGISTRY}/${env.IMAGE_NAME}:${env.IMAGE_TAG}"
                            sh "docker --config /tmp/.docker push ${env.DOCKER_REGISTRY}/${env.IMAGE_NAME}:${env.IMAGE_TAG}"
                        }
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                // Example deployment using docker-compose
                sh "docker-compose down || true" // ignore errors if down fails
                sh "docker-compose up -d"
            }
        }
    }
   post {
        success {
            slackSend channel: "${SLACK_CHANNEL}", message: "Job '${env.JOB_NAME}' build ${env.BUILD_NUMBER} succeeded."
        }
        failure {
            slackSend channel: "${SLACK_CHANNEL}", message: "Job '${env.JOB_NAME}' build ${env.BUILD_NUMBER} failed."
        }
    }
}
