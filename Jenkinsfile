pipeline {
    agent any
    
    tools {
        nodejs 'nodejs-18'  // Must match the name you configured
    }
    
    environment {
        DOCKER_HUB_USERNAME = 'maelhadyf'
        DOCKER_CREDENTIALS = credentials('dockerhub-cred')
        REPO_NAME = 'nodejs-app'
    }
    
    stages {
        stage('Checkout') {
            steps {
                cleanWs()
                checkout scm
                script {
                    def branchName = env.BRANCH_NAME ?: env.GIT_BRANCH ?: 'main'
                    env.BRANCH_NAME = branchName.replaceAll('origin/', '')
                    echo "Building branch: ${env.BRANCH_NAME}"
                }
            }
        }
        
        stage('Install Dependencies') {
            steps {
                // Verify NodeJS installation
                sh 'node --version'
                sh 'npm --version'
                // Install dependencies
                sh 'npm install'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    try {
                        sh """
                        docker build -t ${DOCKER_HUB_USERNAME}/${REPO_NAME}:${env.BRANCH_NAME} .
                        """
                    } catch (Exception e) {
                        error "Failed to build Docker image: ${e.message}"
                    }
                }
            }
        }
        
        stage('Login to DockerHub') {
            steps {
                script {
                    try {
                        sh """
                        echo ${DOCKER_CREDENTIALS_PSW} | docker login -u ${DOCKER_CREDENTIALS_USR} --password-stdin
                        """
                    } catch (Exception e) {
                        error "Failed to login to Docker Hub: ${e.message}"
                    }
                }
            }
        }
        
        stage('Push to DockerHub') {
            steps {
                script {
                    try {
                        sh """
                        docker push ${DOCKER_HUB_USERNAME}/${REPO_NAME}:${env.BRANCH_NAME}
                        """
                    } catch (Exception e) {
                        error "Failed to push image: ${e.message}"
                    }
                }
            }
        }
    }
    
    post {
        always {
            script {
                sh 'docker logout'
                sh """
                docker rmi ${DOCKER_HUB_USERNAME}/${REPO_NAME}:${env.BRANCH_NAME} || true
                """
                cleanWs()
            }
        }
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}
