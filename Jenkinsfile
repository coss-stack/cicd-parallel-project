pipeline {
    agent none

    stages {

        // =====================================================
        // 1. PARALLEL BUILD AND DOCKER CHECK
        // =====================================================
        stage('Parallel Build and Docker') {

            parallel {

                // =================================================
                // AGENT 1 - BUILD / TEST
                // =================================================
                stage('Agent 1 - Build and Test') {

                    agent {
                        label 'linux'
                    }

                    steps {

                        echo '===== AGENT 1 STARTED ====='

                        git branch: 'main',
                            url: 'https://github.com/coss-stack/cicd-parallel-project.git'

                        sh '''
                            echo "Running on Agent 1"
                            echo "Hostname:"
                            hostname

                            echo "Current directory:"
                            pwd

                            echo "Project files:"
                            ls -la

                            echo "App files:"
                            ls -la app

                            echo "Checking HTML file:"
                            cat app/index.html

                            echo "Build/Test completed successfully"
                        '''
                    }
                }


                // =================================================
                // AGENT 2 - DOCKER
                // =================================================
                stage('Agent 2 - Docker') {

                    agent {
                        label 'docker'
                    }

                    steps {

                        echo '===== AGENT 2 STARTED ====='

                        git branch: 'main',
                            url: 'https://github.com/coss-stack/cicd-parallel-project.git'

                        sh '''
                            echo "Running on Agent 2"
                            echo "Hostname:"
                            hostname

                            echo "Current directory:"
                            pwd

                            echo "Project files:"
                            ls -la

                            echo "App files:"
                            ls -la app

                            echo "Docker version:"
                            docker --version

                            echo "Building Docker image..."

                            docker build \
                                -t cicd-app:${BUILD_NUMBER} \
                                ./app

                            echo "Docker images:"
                            docker images
                        '''
                    }
                }
            }
        }


        // =====================================================
        // 2. DOCKER IMAGE
        // =====================================================
        stage('Docker Image Verification') {

            agent {
                label 'docker'
            }

            steps {

                echo '===== DOCKER IMAGE VERIFICATION ====='

                sh '''
                    echo "Docker images available:"
                    docker images

                    echo "Checking image:"
                    docker image inspect cicd-app:${BUILD_NUMBER}

                    echo "Docker image created successfully"
                '''
            }
        }


        // =====================================================
        // 3. ECR LOGIN
        // =====================================================
        stage('ECR Login') {

            agent {
                label 'docker'
            }

            steps {

                sh '''
                    echo "===== ECR LOGIN ====="

                    aws --version

                    aws ecr get-login-password \
                    --region ap-south-1 | \
                    docker login \
                    --username AWS \
                    --password-stdin \
                    726392379799.dkr.ecr.ap-south-1.amazonaws.com

                    echo "ECR login successful"
                '''
            }
        }


        // =====================================================
        // 4. TAG DOCKER IMAGE
        // =====================================================
        stage('Tag Docker Image') {

            agent {
                label 'docker'
            }

            steps {

                sh '''
                    echo "===== TAGGING DOCKER IMAGE ====="

                    docker tag \
                    cicd-app:${BUILD_NUMBER} \
                    726392379799.dkr.ecr.ap-south-1.amazonaws.com/cicd-app:${BUILD_NUMBER}

                    echo "Image tagged successfully"

                    docker images
                '''
            }
        }


        // =====================================================
        // 5. PUSH IMAGE TO ECR
        // =====================================================
        stage('Push Image to ECR') {

            agent {
                label 'docker'
            }

            steps {

                sh '''
                    echo "===== PUSHING IMAGE TO ECR ====="

                    docker push \
                    726392379799.dkr.ecr.ap-south-1.amazonaws.com/cicd-app:${BUILD_NUMBER}

                    echo "Image pushed to ECR successfully"
                '''
            }
        }


        // =====================================================
        // 6. VERIFY ECR
        // =====================================================
        stage('Verify ECR Image') {

            agent {
                label 'docker'
            }

            steps {

                sh '''
                    echo "===== VERIFYING ECR ====="

                    aws ecr describe-images \
                    --repository-name cicd-app \
                    --region ap-south-1

                    echo "ECR verification completed"
                '''
            }
        }


        // =====================================================
        // 7. DEPLOY TO EKS
        // =====================================================
        stage('Deploy to EKS') {

            agent {
                label 'docker'
            }

            steps {

                echo '===== EKS DEPLOYMENT ====='

                git branch: 'main',
                    url: 'https://github.com/coss-stack/cicd-parallel-project.git'

                sh '''
                    echo "Checking Kubernetes files:"
                    ls -la k8s

                    echo "Updating image in Kubernetes deployment..."

                    sed -i \
                    "s|image:.*|image: 726392379799.dkr.ecr.ap-south-1.amazonaws.com/cicd-app:${BUILD_NUMBER}|" \
                    k8s/deployment.yaml

                    echo "Updated deployment file:"
                    cat k8s/deployment.yaml

                    echo "Getting EKS credentials..."

                    aws eks update-kubeconfig \
                    --region ap-south-1 \
                    --name devops-cluster

                    echo "Checking Kubernetes cluster..."

                    kubectl get nodes

                    echo "Applying deployment..."

                    kubectl apply -f k8s/deployment.yaml

                    echo "Applying service..."

                    kubectl apply -f k8s/service.yaml

                    echo "Checking pods..."

                    kubectl get pods

                    echo "Checking service..."

                    kubectl get svc

                    echo "EKS deployment completed successfully"
                '''
            }
        }
    }


    // =========================================================
    // POST ACTIONS
    // =========================================================
    post {

        success {
            echo '========================================'
            echo 'CI/CD PIPELINE SUCCESSFUL'
            echo '========================================'
            echo 'Application deployed successfully'
        }

        failure {
            echo '========================================'
            echo 'CI/CD PIPELINE FAILED'
            echo '========================================'
            echo 'Check the Jenkins console logs'
        }

        always {
            echo 'Pipeline execution completed'
        }
    }
}
