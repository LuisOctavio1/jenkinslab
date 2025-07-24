pipeline {
    agent any
    
    tools {
        nodejs 'NodeJS-7.8.0' // Configurar en Global Tools Configuration
    }
    
    environment {
        // Configurar variables según el branch
        DOCKER_IMAGE = "${env.BRANCH_NAME == 'main' ? 'nodemain' : 'nodedev'}"
        APP_PORT = "${env.BRANCH_NAME == 'main' ? '3000' : '3001'}"
        CONTAINER_PORT = "${env.BRANCH_NAME == 'main' ? '3000:3000' : '3001:3000'}"
        IMAGE_TAG = "v1.0"
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'checkout'
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm install'
            }
        }
        
        stage('Test') {
            steps {
                sh 'npm test'
            }
            post {
                always {
                    publishTestResults testResultsPattern: 'test-results.xml'
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image: ${DOCKER_IMAGE}:${IMAGE_TAG}"
                    sh "docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} ."
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    echo "Deploying ${DOCKER_IMAGE} on port ${APP_PORT}"
                    
                    sh """
                        docker ps -q --filter ancestor=${DOCKER_IMAGE}:${IMAGE_TAG} | xargs -r docker stop
                        
                        docker ps -aq --filter ancestor=${DOCKER_IMAGE}:${IMAGE_TAG} | xargs -r docker rm
                        
                        docker run -d --name ${DOCKER_IMAGE}-container --expose ${APP_PORT} -p ${CONTAINER_PORT} ${DOCKER_IMAGE}:${IMAGE_TAG}
                    """
                    
                    echo "Application deployed successfully on http://localhost:${APP_PORT}"
                }
            }
        }
    }
    
}
