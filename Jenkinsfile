pipeline {

    agent none

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPO = 'cicd-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {

            agent {
                label 'docker-agent'
            }

            steps {
                checkout scm
            }
        }

        stage('Parallel Testing') {

            parallel {

                stage('Unit Test') {

                    agent {
                        label 'docker-agent'
                    }

                    steps {
                        echo 'Running Unit Tests...'

                        sh '''
                            echo "Unit tests started"
                            sleep 5
                            echo "Unit tests completed"
                        '''
                    }
                }

                stage('Code Quality') {

                    agent {
                        label 'scan-agent'
                    }

                    steps {
                        echo 'Running Code Quality Check...'

                        sh '''
                            echo "Code quality analysis started"
                            sleep 5
                            echo "Code quality analysis completed"
                        '''
                    }
                }

                stage('Security Scan') {

                    agent {
                        label 'scan-agent'
                    }

                    steps {
                        echo 'Running Security Scan...'

                        sh '''
                            echo "Security scan started"
                            sleep 5
                            echo "Security scan completed"
                        '''
                    }
                }
            }
        }

        stage('Docker Build') {

            agent {
                label 'docker-agent'
            }

            steps {

                sh '''
                    docker build \
                    -t $ECR_REPO:$IMAGE_TAG \
                    ./app
                '''
            }
        }

        stage('Login to ECR') {

            agent {
                label 'docker-agent'
            }

            steps {

                sh '''
                    aws ecr get-login-password \
                    --region $AWS_REGION | \
                    docker login \
                    --username AWS \
                    --password-stdin \
                    $(aws sts get-caller-identity \
                    --query Account \
                    --output text).dkr.ecr.$AWS_REGION.amazonaws.com
                '''
            }
        }

        stage('Push Docker Image') {

            agent {
                label 'docker-agent'
            }

            steps {

                sh '''
                    ACCOUNT_ID=$(aws sts get-caller-identity \
                    --query Account \
                    --output text)

                    ECR_URL=$ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com

                    docker tag \
                    $ECR_REPO:$IMAGE_TAG \
                    $ECR_URL/$ECR_REPO:$IMAGE_TAG

                    docker push \
                    $ECR_URL/$ECR_REPO:$IMAGE_TAG
                '''
            }
        }

        stage('Deploy to Kubernetes') {

            agent {
                label 'docker-agent'
            }

            steps {

                sh '''
                    ACCOUNT_ID=$(aws sts get-caller-identity \
                    --query Account \
                    --output text)

                    ECR_URL=$ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com

                    sed "s|IMAGE_PLACEHOLDER|$ECR_URL/$ECR_REPO:$IMAGE_TAG|g" \
                    k8s/deployment.yaml > deployment-final.yaml

                    kubectl apply -f deployment-final.yaml

                    kubectl apply -f k8s/service.yaml
                '''
            }
        }

        stage('Verify Deployment') {

            agent {
                label 'docker-agent'
            }

            steps {

                sh '''
                    kubectl get deployments
                    kubectl get pods
                    kubectl get services
                '''
            }
        }
    }

    post {

        success {
            echo 'CI/CD Pipeline completed successfully!'
        }

        failure {
            echo 'CI/CD Pipeline failed!'
        }
    }
}
