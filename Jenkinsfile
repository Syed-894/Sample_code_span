pipeline {
    agent {
        docker {
            image 'docker:latest'
            // Mount the Docker socket and use --privileged (necessary for this approach)
            args '-v /var/run/docker.sock:/var/run/docker.sock --privileged'
        }
    }
    environment {
        DOCKER_REGISTRY = 'syedali161'
        IMAGE_NAME = "jenkins-demo-app"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        SLACK_CHANNEL = '#all-span-devops'
    }
    triggers {
        githubPush()
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
        stage('Build and Push Docker Image') {
            steps {
                script {
                    // 1. Ensure the 'docker' group exists
                    sh 'groupadd -f docker || true'

                    // 2. Add the current user in the container to the 'docker' group
                    sh 'usermod -aG docker $(whoami) || true'

                    // 3. Switch to the current user before building the image
                    sh 'chown -R $(whoami):$(whoami) /var/lib/jenkins || true'
                    sh 'su - $(whoami) -c "docker build -t ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} ."'

                    withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "docker login -u ${DOCKER_USER} -p ${DOCKER_PASSWORD} ${env.DOCKER_REGISTRY}"
                        sh "docker push ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                    }
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
