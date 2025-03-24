pipeline {
    agent any
    
    environment {
        DOCKER_USERNAME = credentials('docker-username')
        DOCKER_PASSWORD = credentials('docker-password')
    }

    stages {
        
        stage('Clone Repository') {
            steps {
                checkout([$class: 'GitSCM', 
                    branches: [[name: '*/main']], 
                    userRemoteConfigs: [[url: 'https://github.com/Syed-894/Sample_code_span.git', 
                    credentialsId: 'github-credentials']]
                ])
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    def imageTag = "syedali161/jenkins-demo-app:${env.BUILD_NUMBER}"
                    sh "docker build -t ${imageTag} ."
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry([credentialsId: 'docker', url: 'https://index.docker.io/v1/']) {
                        sh '''
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                        docker tag syedali161/jenkins-demo-app:${env.BUILD_NUMBER} syedali161/jenkins-demo-app:latest
                        docker push syedali161/jenkins-demo-app:${env.BUILD_NUMBER}
                        docker push syedali161/jenkins-demo-app:latest
                        '''
                    }
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
    }
}
