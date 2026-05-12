pipeline {
    agent any

    tools {
        nodejs 'nodejs-18'
    }

    environment {
        APP_NAME = 'hello-jenkins'
        JEST_JUNIT_OUTPUT_DIR = 'test-results'
        JEST_JUNIT_OUTPUT_NAME = 'junit.xml'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                echo "Branch: ${GIT_BRANCH} | Commit: ${GIT_COMMIT[0..7]}"
            }
        }

        stage('Install') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
            post {
                always {
                    junit 'test-results/junit.xml'
                }
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'app.js,package.json', fingerprint: true
            }
        }

    }

    post {
        success { echo "Build ${BUILD_NUMBER} succeeded." }
        failure { echo "Build ${BUILD_NUMBER} failed."   }
    }
}