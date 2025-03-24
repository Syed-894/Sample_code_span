pipeline {
    agent {
        docker {
            image 'docker:latest'

            args '-u root'

        }
    }
    environment {
        DOCKER_REGISTRY = 'syedali161'  
        IMAGE_NAME = "jenkins-demo-app"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        SLACK_CHANNEL = '#all-span-devops'       // Replace with your Slack channel
    }

        triggers {
        githubPush() // The githubPush trigger does not take branchFilter.  It triggers on any push.
    }
    parameters {
        string(
            name: 'GIT_BRANCH',
            defaultValue: 'main',
            description: 'Branch to checkout'
        )
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: "${params.GIT_BRANCH}",
                    url: 'https://github.com/Syed-894/Sample_code_span.git' 
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    def dockerImage = docker.build("${env.IMAGE_NAME}:${env.IMAGE_TAG}", '.')
                    dockerImage.push()
                }
                withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh "docker login -u ${DOCKER_USER} -p ${DOCKER_PASSWORD} ${env.DOCKER_REGISTRY}"
                    sh "docker push ${env.DOCKER_REGISTRY}/${env.IMAGE_NAME}:${env.IMAGE_TAG}"
                }
            }
        }
        stage('Deploy') {
            steps {

                sh "docker-compose down || true"  
                sh "docker-compose up -d"
            }
        }
    }
    post {
        success {
            slackSend channel: "${SLACK_CHANNEL}",
                message: "Job '${env.JOB_NAME}' build ${env.BUILD_NUMBER} succeeded."
        }
        failure {
            slackSend channel: "${SLACK_CHANNEL}",
                message: "Job '${env.JOB_NAME}' build ${env.BUILD_NUMBER} failed."
        }
    }
}
