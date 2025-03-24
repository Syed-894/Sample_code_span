pipeline {
    agent {
        docker {
            // Use a Docker image that includes common utilities
            image 'alpine/git:latest'  // Alpine with Git, lightweight and has shell utils
            // Mount the Docker socket and use --privileged (necessary for DinD)
            args '-v /var/run/docker.sock:/var/run/docker.sock --privileged'
        }
    }
    environment {
        DOCKER_REGISTRY = 'syedali161'
        IMAGE_NAME      = "jenkins-demo-app"
        IMAGE_TAG       = "${env.BUILD_NUMBER}"
        SLACK_CHANNEL   = '#all-span-devops'
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
                    sh 'apk add --no-cache shadow'  // Install shadow for groupadd, usermod
                    sh 'groupadd -f docker || true'

                    // 2. Determine the UID and GID of the 'jenkins' user on the host
                    def hostUid = sh(returnStdout: true, script: 'id -u jenkins').trim()
                    def hostGid = sh(returnStdout: true, script: 'id -g jenkins').trim()

                    // 3. Create a 'jenkins' user inside the container with the same UID/GID
                    sh "adduser -D -u ${hostUid} -g ${hostGid} jenkins"

                    // 4. Add the 'jenkins' user to the 'docker' group inside the container
                    sh 'usermod -aG docker jenkins'

                    // 5. Switch to the 'jenkins' user before building the image
                    sh 'chown -R jenkins:jenkins /var/lib/jenkins'
                    sh 'su - jenkins -c "docker build -t ${env.IMAGE_NAME}:${env.IMAGE_TAG} ."'

                    // Push the image
                    withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "docker login -u ${DOCKER_USER} -p ${DOCKER_PASSWORD} ${env.DOCKER_REGISTRY}"
                        sh "docker push ${env.DOCKER_REGISTRY}/${env.IMAGE_NAME}:${env.IMAGE_TAG}"
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

