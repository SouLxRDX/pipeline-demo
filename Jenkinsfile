pipeline {
    agent any

    parameters {
        string(name: 'VERSION', defaultValue: '1.0.0', description: 'Version to build')
        choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'prod'], description: 'Target environment')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run tests?')
    }

    environment {
        APP_NAME = 'pipeline-demo'
        DEPLOY_DIR = "deploy/${params.ENVIRONMENT}"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Branch: ${env.GIT_BRANCH}"
                echo "Build: #${env.BUILD_NUMBER}"
                echo "Version: ${params.VERSION}"
                echo "Environment: ${params.ENVIRONMENT}"
                echo "Run Tests: ${params.RUN_TESTS}"
            }
        }

        stage('Build') {
            steps {
                echo "Building version ${params.VERSION} for ${params.ENVIRONMENT}..."
                sh '''
                    mkdir -p build
                    echo "App: $APP_NAME" > build/output.txt
                    echo "Version: $VERSION" >> build/output.txt
                    echo "Environment: $ENVIRONMENT" >> build/output.txt
                    cat build/output.txt
                '''
            }
        }

        stage('Test') {
            when {
                expression { params.RUN_TESTS == true }
            }
            steps {
                echo 'Running tests...'
                sh '''
                    test -f build/output.txt && echo "PASS: file exists" || echo "FAIL"
                    grep "Version" build/output.txt && echo "PASS: version found" || echo "FAIL"
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploying to ${params.ENVIRONMENT}..."
                sh '''
                    mkdir -p deploy/$ENVIRONMENT
                    cp build/output.txt deploy/$ENVIRONMENT/
                    echo "Deployed version $VERSION to $ENVIRONMENT"
                '''
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts artifacts: 'build/**,deploy/**', fingerprint: true
            }
        }
    }

    post {
        success {
            echo "SUCCESS: ${APP_NAME} v${params.VERSION} deployed to ${params.ENVIRONMENT}!"
        }
        failure {
            echo "FAILED: Build #${BUILD_NUMBER} failed!"
        }
        always {
            cleanWs()
        }
    }
}
