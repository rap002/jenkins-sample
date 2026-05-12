pipeline {
    agent any

    tools {
        nodejs 'nodejs-18'
    }

    environment {
        APP_NAME = 'hello-jenkins'
        JEST_JUNIT_OUTPUT_DIR  = 'test-results'
        JEST_JUNIT_OUTPUT_NAME = 'junit.xml'
    }

    stages {

        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Install') {
            steps { sh 'npm ci' }
        }

        stage('Test') {
            steps { sh 'npm test' }
            post {
                always { junit 'test-results/junit.xml' }
            }
        }

        stage('Build Image') {
            steps {
                sh "docker build -t ${APP_NAME}:${BUILD_NUMBER} ."
            }
        }

        stage('Run Container') {
            steps {
                sh "docker rm -f ${APP_NAME} || true"
                sh "docker run -d --name ${APP_NAME} -p 3001:3000 ${APP_NAME}:${BUILD_NUMBER}"
                sh "sleep 3 && curl -f http://localhost:3001/ || (docker logs ${APP_NAME} && exit 1)"
            }
        }

    }

    post {
        success { echo "App running at http://localhost:3001/" }
        failure { echo "Build ${BUILD_NUMBER} failed." }
    }
}