pipeline {
    agent any
    
    environment {
        DOCKER_HUB_CREDENTIALS = credentials('dockerhub-credentials')
        DOCKER_IMAGE = 'vijju15/flask-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code...'
                checkout scm
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    echo 'Building Docker image...'
                    bat "docker build -t %DOCKER_IMAGE%:${IMAGE_TAG} ."
                    bat "docker tag %DOCKER_IMAGE%:${IMAGE_TAG} %DOCKER_IMAGE%:latest"
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                script {
                    echo 'Logging in to Docker Hub...'
                    bat "echo %DOCKER_HUB_CREDENTIALS_PSW% | docker login -u %DOCKER_HUB_CREDENTIALS_USR% --password-stdin"
                    
                    echo 'Pushing Docker image...'
                    bat "docker push %DOCKER_IMAGE%:${IMAGE_TAG}"
                    bat "docker push %DOCKER_IMAGE%:latest"
                    
                    echo 'Logging out from Docker Hub...'
                    bat "docker logout"
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo 'Deploying to Kubernetes...'
                    
                    // Update the image in deployment
                    bat """
                        kubectl set image deployment/flask-app-deployment ^
                        flask-app=%DOCKER_IMAGE%:${IMAGE_TAG} ^
                        --record
                    """
                    
                    // Check rollout status
                    bat "kubectl rollout status deployment/flask-app-deployment"
                    
                    echo 'Deployment completed successfully!'
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                script {
                    echo 'Verifying deployment...'
                    bat "kubectl get pods -l app=flask-app"
                    bat "kubectl get service flask-app-service"
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
        always {
            echo 'Cleaning up...'
            bat "docker system prune -f"
        }
    }
}