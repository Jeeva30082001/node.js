pipeline {
    agent any

    environment {
        // Docker image
        DOCKER_IMAGE = 'jeeva3008/backend:latest'

        // Docker Hub
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials'

        // SSH
        SSH_CREDENTIALS_ID = 'server2-ssh-key'
        SERVER2_USER = 'ubuntu'
        SERVER2_HOST = 'SERVER2_PUBLIC_IP'

        // App
        CONTAINER_NAME = 'node_backend'
        APP_PORT = '3000'

        // Mongo (running already on Server-2)
        MONGO_URI = 'mongodb://admin:admin123@mongo_db:27017/todo_db?authSource=admin'
        MONGO_DB = 'todo_db'
        MONGO_COLLECTION = 'tasks'
    }

    stages {

        stage('Clone Repository') {
            steps {
                echo '📥 Cloning repository'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '🐳 Building Docker image'
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: DOCKER_CREDENTIALS_ID,
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                echo '📤 Pushing image to Docker Hub'
                sh "docker push ${DOCKER_IMAGE}"
            }
        }

        stage('Deploy to Server-2 via SSH') {
            steps {
                echo '🚀 Deploying application on Server-2'
                sshagent(credentials: [SSH_CREDENTIALS_ID]) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${SERVER2_USER}@${SERVER2_HOST} << EOF
                      docker pull ${DOCKER_IMAGE}
                      docker rm -f ${CONTAINER_NAME} || true
                      docker run -d \\
                        --name ${CONTAINER_NAME} \\
                        -p ${APP_PORT}:${APP_PORT} \\
                        -e PORT=${APP_PORT} \\
                        -e MONGO_URI=${MONGO_URI} \\
                        -e MONGO_DB=${MONGO_DB} \\
                        -e MONGO_COLLECTION=${MONGO_COLLECTION} \\
                        ${DOCKER_IMAGE}
                    EOF
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ DEPLOYMENT SUCCESSFUL'
        }
        failure {
            echo '❌ DEPLOYMENT FAILED'
        }
        always {
            sh 'docker logout || true'
        }
    }
}
