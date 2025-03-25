pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = 'syedali161'                                      // DockerHub username
        IMAGE_NAME = "jenkins-demo-app"                                         // Docker Image name
        IMAGE_TAG = "${env.BUILD_NUMBER}"                                        // Build number as image tag
        DOCKER_CREDENTIALS = 'docker-credentials'                                         // Jenkins Credentials ID
        SLACK_CHANNEL = '#all-span-devops'                                     // Slack channel name
        GIT_REPO_URL = 'https://github.com/Syed-894/Sample_code_span.git'  // GitHub Repo U
    }

    triggers {
        githubPush()  // Trigger on GitHub push
    }

    parameters {
        string(
            name: 'GIT_BRANCH',
            defaultValue: 'main',
            description: 'Branch to checkout'
        )
    }

    stages {

        stage('Clone GitHub Repo') {
            steps {
                git branch: "${params.GIT_BRANCH}",
                    url: "${GIT_REPO_URL}"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh """
                            docker build -t ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} .
                        """
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh """
                            docker push ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                        """
                    }
                }
            }
        }

        stage('Deploy with Docker Compose') {
            steps {
                sh """
                    docker-compose down || true
                    docker-compose up -d
                """
            }
        }
    }

    post {
        success {
            slackSend(
                channel: "${SLACK_CHANNEL}",
                color: 'good',
                message: "✅ SUCCESS: Build #${env.BUILD_NUMBER} completed successfully! 🎉"
            )
        }
        failure {
            slackSend(
                channel: "${SLACK_CHANNEL}",
                color: 'danger',
                message: "❌ FAILURE: Build #${env.BUILD_NUMBER} failed. Please check the logs. 🚨"
            )
        }
    }
}
