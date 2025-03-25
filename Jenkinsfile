pipeline {
    agent {
        docker {
            image 'alpine/git:latest'
            // Mount the Docker socket and use --privileged (necessary for this approach)
            args '-v /var/run/docker.sock:/var/run/docker.sock --privileged'
        }
    }
    //Move SLACK_CHANNEL definition here
    environment {
        DOCKER_REGISTRY = 'syedali161'
        IMAGE_NAME = "jenkins-demo-app"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        
        GIT_BRANCH = 'main' //Added default
        GIT_REPO_URL = 'https://github.com/Syed-894/Sample_code_span.git'
        DOCKER_CREDENTIALS = 'docker-credentials'
        
    }
    string SLACK_CHANNEL = '#all-span-devops'  // Slack channel name - defined outside stages


    triggers {
        githubPush()
    }

    parameters {
        string(
            name: 'GIT_BRANCH',
            defaultValue: GIT_BRANCH,
            description: 'Branch to checkout'
        )
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${params.GIT_BRANCH}",
                    url: "${GIT_REPO_URL}"
            }
        }
        stage('Build and Push Docker Image') {
            steps {
                script {
                    // 1. Install necessary tools
                    sh 'apk add --no-cache shadow coreutils'

                    // 2. Ensure the 'docker' group exists
                    sh 'groupadd -f docker || true'

                    // 3. Add the current user to the 'docker' group
                    sh 'usermod -aG docker $(whoami) || true'

                    // 4. Switch to the current user before building the image
                    sh 'chown -R $(whoami):$(whoami) /var/lib/jenkins || true'
                    sh 'su - $(whoami) -c "docker build -t ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} ."'

                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASSWORD')]) {
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
            slackSend channel: SLACK_CHANNEL,
                message: "Job '${env.JOB_NAME}' build ${env.BUILD_NUMBER} succeeded."
        }
        failure {
            slackSend channel: SLACK_CHANNEL,
                message: "Job '${env.JOB_NAME}' build ${env.BUILD_NUMBER} failed."
        }
    }
}
