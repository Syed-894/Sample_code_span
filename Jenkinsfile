pipeline {
    agent { docker { image 'docker:latest' } }
    environment {
        DOCKER_REGISTRY = 'syedali161'  //  your DockerHub username
        IMAGE_NAME = "jenkins-demo-app"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        SLACK_CHANNEL = '#all-span-devops'  // your Slack channel name
    }
    triggers {
        githubPush(branchFilter: 'main')
    }
    parameters {
        string(name: 'GIT_BRANCH', defaultValue: 'main', description: 'Branch to checkout')
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: "${params.GIT_BRANCH}", url: 'https://github.com/Syed-894/Sample_code_span.git' // your github username
            }
        }
        stage('Build and Push Docker Image') {
            steps {
                script {
                    def dockerImage = docker.build("${env.IMAGE_NAME}:${env.IMAGE_TAG}", '.')
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASSWORD')]) {
                         sh "docker login -u ${DOCKER_USER} -p ${DOCKER_PASSWORD} ${env.DOCKER_REGISTRY}"
                         dockerImage.push("${env.DOCKER_REGISTRY}/${env.IMAGE_NAME}:${env.IMAGE_TAG}")
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                //  Example deployment using docker-compose
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
