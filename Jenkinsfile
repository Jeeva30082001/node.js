pipeline {
    agent any

    environment {
        // Docker image
        DOCKER_IMAGE = 'jeeva3008/backend:latest'

        // Docker Hub credentials
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials'

        // SSH to Server-2
        SSH_CREDENTIALS_ID = 'server2-ssh-key'
        SERVER2_USER = 'ubuntu'
        SERVER2_HOST = '100.25.213.78'

        // App
        CONTAINER_NAME = 'node_backend'
        APP_PORT = '3000'

        // Mongo (already running in appnet on Server-2)
        MONGO_URI = 'mongodb://admin:admin123@mongo_db:27017/todo_db?authSource=admin'
        MONGO_DB = 'todo_db'
        MONGO_COLLECTION = 'tasks'
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo '📥 Checking out source code'
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
                echo '🔐 Logging into Docker Hub'
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

        stage('Deploy Backend to Server-2') {
            steps {
                echo '🚀 Deploying backend on Server-2'
                sshagent(credentials: [SSH_CREDENTIALS_ID]) {
                    sh '''
ssh -o StrictHostKeyChecking=no ubuntu@100.25.213.78 << 'EOF'
docker pull jeeva3008/backend:latest
docker rm -f node_backend || true

docker run -d \
  --name node_backend \
  --network appnet \
  --restart unless-stopped \
  -p 3000:3000 \
  -e PORT=3000 \
  -e MONGO_URI="mongodb://admin:admin123@mongo_db:27017/todo_db?authSource=admin" \
  -e MONGO_DB=todo_db \
  -e MONGO_COLLECTION=tasks \
  jeeva3008/backend:latest
EOF
'''
                }
            }
        }
    }

    post {
        success {
            echo '✅ BACKEND DEPLOYMENT SUCCESSFUL'
        }
        failure {
            echo '❌ BACKEND DEPLOYMENT FAILED'
        }
        always {
            sh 'docker logout || true'
        }
    }
}
