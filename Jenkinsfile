pipeline {
    agent any

    parameters {
        string(name: 'VERSION', defaultValue: '1.0.0', description: 'Image version')
        choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'prod'], description: 'Target environment')
    }

    environment {
        APP_NAME = 'pipeline-demo'
        DOCKER_HUB = credentials('dockerhub-creds')
        IMAGE_NAME = "${DOCKER_HUB_USR}/${APP_NAME}:${params.VERSION}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                echo "Building: ${IMAGE_NAME}"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME} ."
            }
        }

        stage('Test Image') {
            steps {
                sh '''
                docker run --rm ${IMAGE_NAME} python -c "print('Container test passed!')"
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sh '''
                echo "$DOCKER_HUB_PSW" | docker login -u "$DOCKER_HUB_USR" --password-stdin
                docker push ${IMAGE_NAME}
                '''
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                docker stop ${APP_NAME} || true
                docker rm ${APP_NAME} || true
                docker run -d --name ${APP_NAME} -p 8000:8000 ${IMAGE_NAME}
                '''
            }
        }

        stage('Cleanup') {
            steps {
                sh 'docker logout || true'
                cleanWs()
            }
        }
    }

    post {
        success {
            echo "SUCCESS: ${IMAGE_NAME} built and deployed!"
        }
        failure {
            echo "FAILED: Build #${BUILD_NUMBER} failed!"
        }
    }
}
