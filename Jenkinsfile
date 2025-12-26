pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'jeeva3008/backend:latest'
        GIT_REPO = 'https://github.com/Jeeva30082001/Node.js_docker_yml_jenkins_mangoDB.git'
        GIT_BRANCH = 'main'
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials'

        CONTAINER_NAME = 'node_backend'
        DOCKER_NETWORK = 'appnet'

        // App environment values
        APP_PORT = '3000'
        MONGO_URI = 'mongodb://admin:admin123@mongo_db:27017/todo_db?authSource=admin'
        MONGO_DB = 'todo_db'
        MONGO_COLLECTION = 'tasks'
    }

    stages {

        stage('Clone Repository') {
            steps {
                echo '📥 Cloning GitHub repository...'
                git branch: "${GIT_BRANCH}", url: "${GIT_REPO}"
            }
        }

        stage('Ensure Docker Network') {
            steps {
                echo '🌐 Ensuring Docker network exists...'
                sh '''
                docker network inspect appnet >/dev/null 2>&1 || \
                docker network create appnet
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '🐳 Building Docker image...'
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('Login to Docker Hub') {
            steps {
                echo '🔐 Logging into Docker Hub...'
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKER_CREDENTIALS_ID}",
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    sh '''
                    echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                echo '📤 Pushing image to Docker Hub...'
                sh "docker push ${DOCKER_IMAGE}"
            }
        }

        stage('Deploy Docker Container') {
            steps {
                echo '🚀 Deploying Node.js backend container...'
                sh '''
                docker pull ${DOCKER_IMAGE}
                docker rm -f ${CONTAINER_NAME} || true

                docker run -d \
                  --name ${CONTAINER_NAME} \
                  --network ${DOCKER_NETWORK} \
                  -e PORT=${APP_PORT} \
                  -e MONGO_URI=${MONGO_URI} \
                  -e MONGO_DB=${MONGO_DB} \
                  -e MONGO_COLLECTION=${MONGO_COLLECTION} \
                  -p 3000:3000 \
                  ${DOCKER_IMAGE}
                '''
            }
        }
    }

    post {
        success {
            echo '✅ DEPLOYMENT SUCCESSFUL'
            echo '🚀 Node.js backend is live on port 3000'
        }

        failure {
            echo '❌ DEPLOYMENT FAILED'
            echo '⚠️ Please check Jenkins console logs for errors'
        }

        always {
            echo '🧹 Cleaning up Docker login'
            sh 'docker logout || true'
        }
    }
}
