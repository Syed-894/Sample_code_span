pipeline {
    agent {
        docker {
            image 'docker:latest'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
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
                git branch: "${params.GIT_BRANCH}", url: 'https://github.com/Syed-894/Sample_code_span.git' // Your GitHub username
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    // Fix docker permissions by creating .docker in workspace and using GID
                    sh 'mkdir -p ${WORKSPACE}/.docker'
                    sh 'chown -R $USER:0 ${WORKSPACE}/.docker' // Use GID 0 (root group)
                    def dockerImage = docker.build("${env.IMAGE_NAME}:${env.IMAGE_TAG}", '.')
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "docker login -u $DOCKER_USER -p $DOCKER_PASSWORD ${env.DOCKER_REGISTRY}"
                        dockerImage.push("${env.DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}")
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
