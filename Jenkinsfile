pipeline {
    agent any
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-cred')
        DOCKER_IMAGE = 'your-dockerhub-username/nodejs-app'
    }
    
    stages {
        stage('Checkout') {
            steps {
                script {
                    // Get branch name
                    env.BRANCH_NAME = env.BRANCH_NAME ?: 'main'
                    echo "Building branch: ${env.BRANCH_NAME}"
                }
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    // Build image with branch name as tag
                    sh "docker build -t ${DOCKER_IMAGE}:${env.BRANCH_NAME} ."
                }
            }
        }
        
        stage('Login to DockerHub') {
            steps {
                script {
                    // Login to DockerHub
                    sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                }
            }
        }
        
        stage('Push to DockerHub') {
            steps {
                script {
                    // Push image with branch tag
                    sh "docker push ${DOCKER_IMAGE}:${env.BRANCH_NAME}"
                }
            }
        }
    }
    
    post {
        always {
            // Cleanup
            sh 'docker logout'
            cleanWs()
        }
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}
