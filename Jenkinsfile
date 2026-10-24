pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        AWS_REGION = 'us-east-1'
        FRONTEND_REPO = 'streaming-frontend'
        HELLO_REPO = 'streaming-hello'
        PROFILE_REPO = 'streaming-profile'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            steps {
                echo 'Checking application package files'

                sh '''
                    set -eu
                    test -f frontend/package.json
                    test -f backend/helloService/package.json
                    test -f backend/profileService/package.json
                '''
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                    set -eu

                    docker build \
                      -t streaming-frontend:local ./frontend

                    docker build \
                      -f backend/Dockerfile \
                      --build-arg SERVICE_NAME=helloService \
                      -t streaming-hello:local ./backend

                    docker build \
                      -f backend/Dockerfile \
                      --build-arg SERVICE_NAME=profileService \
                      -t streaming-profile:local ./backend
                '''
            }
        }

        stage('Login to Amazon ECR') {
            steps {
                sh '''
                    set -eu

                    ACCOUNT_ID=$(aws sts get-caller-identity \
                      --query Account --output text)

                    REGISTRY="${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"

                    aws ecr get-login-password \
                      --region "$AWS_REGION" |
                      docker login \
                        --username AWS \
                        --password-stdin "$REGISTRY"
                '''
            }
        }

        stage('Push Images to ECR') {
            steps {
                sh '''
                    set -eu

                    ACCOUNT_ID=$(aws sts get-caller-identity \
                      --query Account --output text)

                    REGISTRY="${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
                    TAG=$(git rev-parse HEAD)

                    docker tag streaming-frontend:local \
                      "$REGISTRY/$FRONTEND_REPO:$TAG"

                    docker tag streaming-hello:local \
                      "$REGISTRY/$HELLO_REPO:$TAG"

                    docker tag streaming-profile:local \
                      "$REGISTRY/$PROFILE_REPO:$TAG"

                    docker push "$REGISTRY/$FRONTEND_REPO:$TAG"
                    docker push "$REGISTRY/$HELLO_REPO:$TAG"
                    docker push "$REGISTRY/$PROFILE_REPO:$TAG"
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: All three images were pushed to ECR.'
        }

        failure {
            echo 'FAILED: Check Console Output for the error.'
        }

        always {
            sh 'docker logout || true'
        }
    }
}