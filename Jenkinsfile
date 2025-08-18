pipeline {
    agent any

    environment {
        REGISTRY = 'jeeva1306'
        IMAGE_NAME = 'myapp'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        DOCKER_CRED_ID = 'dockerhub-creds'
        K8S_MANIFEST_PATH = 'k8s'  // folder where deployment.yaml & service.yaml are kept
        AWS_CRED_ID = 'aws-eks-creds' // Added for AWS access
        AWS_REGION = 'us-east-1'    // EKS region
        EKS_CLUSTER = 'myeks'         // EKS cluster name
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'master', url: 'https://github.com/jeeva113/gittest.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                script {
                    docker.withRegistry("https://index.docker.io/v1/", "${DOCKER_CRED_ID}") {
                        docker.image("${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}").push()
                    }
                }
            }
        }

        stage('Update K8s Manifests') {
            steps {
                sh '''
                  sed -i "s#jeeva1306/myapp:.*#jeeva1306/myapp:${IMAGE_TAG}#g" ${K8S_MANIFEST_PATH}/deployment.yaml
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                // Use AWS credentials safely
                withAWS(credentials: "${AWS_CRED_ID}", region: "${AWS_REGION}") {
                    sh '''
                      echo "🚀 Updating kubeconfig..."
                      aws eks update-kubeconfig --name ${EKS_CLUSTER}

                      echo "🚀 Deploying to EKS..."
                      kubectl apply -f ${K8S_MANIFEST_PATH}/deployment.yaml
                      kubectl apply -f ${K8S_MANIFEST_PATH}/service.yaml

                      echo "🚀 Waiting for rollout to complete..."
                      kubectl rollout status deployment/calculator-deployment
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ App deployed successfully! Check ELB via 'kubectl get svc calculator-service'"
        }
        failure {
            echo "❌ Deployment failed!"
        }
    }
