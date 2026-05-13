pipeline {
    agent any

    tools {
        nodejs 'nodejs-18'
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
        IMAGE_NAME      = "foofa/hello-jenkins"
        IMAGE_TAG       = "${BUILD_NUMBER}"
        DEPLOYMENT_NAME = "hello-jenkins"
        CONTAINER_NAME  = "hello-jenkins"
        K8S_NAMESPACE   = "default"
        JEST_JUNIT_OUTPUT_DIR  = 'test-results'
        JEST_JUNIT_OUTPUT_NAME = 'junit.xml'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/rap002/jenkins-sample'
                echo "Building ${GIT_BRANCH} @ ${GIT_COMMIT[0..7]}"
            }
        }

        stage('Install & Test') {
            steps { sh 'npm ci && npm test' }
            post {
                always { junit 'test-results/junit.xml' }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                sh "docker tag  ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest"
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sh """
                    echo ${DOCKERHUB_CREDENTIALS_PSW} | \
                    docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin
                    docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    docker push ${IMAGE_NAME}:latest
                """
            }
        }

        stage('Deploy to Kubernetes') {
            when { branch 'main' }
            steps {
                sh """
                    kubectl set image deployment/${DEPLOYMENT_NAME} \
                        ${CONTAINER_NAME}=${IMAGE_NAME}:${IMAGE_TAG} \
                        -n ${K8S_NAMESPACE}

                    kubectl rollout status deployment/${DEPLOYMENT_NAME} \
                        -n ${K8S_NAMESPACE} \
                        --timeout=120s
                """
            }
        }

        stage('Verify') {
            when { branch 'main' }
            steps {
                sh "kubectl get pods -n ${K8S_NAMESPACE} -l app=${DEPLOYMENT_NAME}"
                sh "kubectl get svc hello-jenkins-svc -n ${K8S_NAMESPACE}"
            }
        }

        stage('Cleanup') {
            steps { sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true" }
        }

    }

    post {
        success {
            echo "Build ${BUILD_NUMBER}: image :${IMAGE_TAG} deployed to Kubernetes."
        }
        failure {
            sh """
                echo 'Pipeline failed — rolling back Kubernetes deployment'
                kubectl rollout undo deployment/${DEPLOYMENT_NAME} \
                    -n ${K8S_NAMESPACE} || true
            """
        }
    }
}