pipeline {

    agent any

    environment {
        AWS_REGION    = 'ap-south-1'
        AWS_ACCOUNT   = '504895205437'
        ECR_REPOSITORY = 'devops-react-app'
        ECR_REGISTRY  = '504895205437.dkr.ecr.ap-south-1.amazonaws.com'

        IMAGE_NAME    = 'devops-react-app'
        IMAGE_TAG     = "${BUILD_NUMBER}"

        CONTAINER_NAME = 'devops-react-container'
        APP_PORT       = '8081'
    }

    stages {

        stage('Checkout Source Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('react-basics-notes-todo-main') {
                    sh '''
                        set -e

                        docker build \
                            -t ${IMAGE_NAME}:${IMAGE_TAG} .
                    '''
                }
            }
        }

        stage('Login to AWS ECR') {
            steps {
                sh '''
                    set -e

                    aws ecr get-login-password \
                        --region ${AWS_REGION} | \
                    docker login \
                        --username AWS \
                        --password-stdin ${ECR_REGISTRY}
                '''
            }
        }

        stage('Tag and Push Image') {
            steps {
                sh '''
                    set -e

                    docker tag \
                        ${IMAGE_NAME}:${IMAGE_TAG} \
                        ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}

                    docker push \
                        ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}
                '''
            }
        }

        stage('Deploy to EC2') {
            steps {
                sh '''
                    set -e

                    docker pull \
                        ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}

                    if docker ps -a --format '{{.Names}}' | grep -qx "${CONTAINER_NAME}"; then
                        docker rm -f ${CONTAINER_NAME}
                    fi

                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p ${APP_PORT}:80 \
                        --restart unless-stopped \
                        ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}

                    sleep 5

                    curl -f http://localhost:${APP_PORT}/react-basics-notes-todo/
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully!'
            echo "Docker Image: ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}"
        }

        failure {
            echo 'CI/CD Pipeline failed. Please check the Jenkins console output.'
        }
    }
}
