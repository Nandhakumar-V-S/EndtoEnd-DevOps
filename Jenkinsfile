pipeline {

    agent any

    environment {
        AWS_REGION     = 'ap-south-1'
        AWS_ACCOUNT    = '504895205437'
        ECR_REPOSITORY = 'devops-react-app'
        ECR_REGISTRY   = '504895205437.dkr.ecr.ap-south-1.amazonaws.com'

        IMAGE_NAME     = 'devops-react-app'
        IMAGE_TAG      = "${BUILD_NUMBER}"

        CONTAINER_NAME = 'devops-react-container'
        APP_PORT       = '8081'
    }

    stages {

        stage('Checkout Source Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Nandhakumar-V-S/EndtoEnd-DevOps.git'
            }
        }
        
        stage('Validate Environment and Tools') {
            steps {
                sh '''
                    set -e
                    echo "Checking installed tools..."

                    git --version
                    docker --version
                    aws --version
                    
                    echo "Checking AWS identity..."
                    ACTUAL_ACCOUNT=$(aws sts get-caller-identity \
                        --query Account \
                        --output text)
                    echo "AWS Account: $ACTUAL_ACCOUNT"
                    if [ "$ACTUAL_ACCOUNT" != "$AWS_ACCOUNT" ]; then
                        echo "ERROR: AWS account mismatch!"
                        exit 1
                    fi
                    echo "Environment validation successful."
                '''
            }
        }
        
        stage('Build Docker Image') {
            steps {
                dir('react-basics-notes-todo-main') {
                    sh '''
                        set -e
                        docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                    '''
                }
            }
        }
        
         stage('Validate Docker Image') {
             steps {
                 sh '''
                     set -e
                     echo "Checking Docker image..."
                     docker image inspect \
                         ${IMAGE_NAME}:${IMAGE_TAG} > /dev/null
                     docker image ls ${IMAGE_NAME}
                     echo "Docker image validation successful."
                 '''
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

        stage('Push Docker Image to ECR') {
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

        stage('Deploy Application') {
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
        
        stage('Deployment Summary') {
            steps {
                sh '''
                   echo "       DEPLOYMENT SUMMARY"
                   
                    echo "Application: ${IMAGE_NAME}"
                    echo "Build Number: ${BUILD_NUMBER}"
                    echo "Docker Image: ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}"
                    echo "Container: ${CONTAINER_NAME}"
                    echo "Application Port: ${APP_PORT}"
                    echo "Application URL: ${APP_URL}"

                  
                    docker ps --filter name=${CONTAINER_NAME}
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully!'
            echo "Deployed image: ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}"
        }

        failure {
            echo 'CI/CD Pipeline failed. Check Console Output.'
        }
    }
}