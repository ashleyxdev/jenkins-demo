pipeline {
    agent any

    environment {
        IMAGE_NAME = 'content-manager'
        CONTAINER_NAME = 'content-manager-app'
        APP_PORT = '5000'
        DATA_DIR = '/tmp/test-data'
    }

    stages {

        stage('Checkout') {
            steps {
                echo '📥 Pulling latest code from GitHub...'
                checkout scm
            }
        }

        stage('Install') {
            steps {
                echo '📦 Installing dependencies...'
                sh 'node --version'
                sh 'npm --version'
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                echo '🧪 Running tests...'
                sh 'DATA_DIR=$DATA_DIR npm test'
            }
        }

        stage('Build') {
            steps {
                echo '🔨 Building Docker image...'
                sh 'docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .'
                sh 'docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest'
                echo "✅ Image built: ${IMAGE_NAME}:${BUILD_NUMBER}"
            }
        }

        stage('Deploy') {
            steps {
                echo '🚀 Deploying application...'
                sh '''
                    if [ $(docker ps -aq -f name=${CONTAINER_NAME}) ]; then
                        echo "Stopping old container..."
                        docker stop ${CONTAINER_NAME}
                        docker rm ${CONTAINER_NAME}
                    fi
                '''
                sh '''
                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p ${APP_PORT}:5000 \
                        -v content-data:/data \
                        ${IMAGE_NAME}:latest
                '''
                echo "✅ App deployed at http://localhost:${APP_PORT}"
            }
        }

    }

    post {
        success {
            echo '🎉 Pipeline completed! App is live at http://localhost:5000'
        }
        failure {
            echo '❌ Pipeline failed! Check the logs above.'
            sh '''
                if [ $(docker ps -aq -f name=${CONTAINER_NAME}) ]; then
                    docker stop ${CONTAINER_NAME}
                    docker rm ${CONTAINER_NAME}
                fi
            '''
        }
        always {
            echo '🧹 Cleaning up test data...'
            sh 'rm -rf $DATA_DIR'
        }
    }
}
