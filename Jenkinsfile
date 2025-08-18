pipeline {
    agent any

    environment {
        REGISTRY = 'jeeva1306'
        IMAGE_NAME = 'myapp'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        DOCKER_CRED_ID = 'dockerhub-creds'       // Jenkins DockerHub credentials ID
        AWS_REGION = 'us-east-1'
        AWS_CRED_ID = 'aws-creds-id'            // Jenkins AWS credentials ID
        K8S_MANIFEST_PATH = "${WORKSPACE}/k8s"  // Path to your k8s manifests
        KUBECTL_PATH = '/root/bin/kubectl'      // Absolute path to kubectl
    }

    stages {
        stage('Checkout SCM') {
            steps {
                git branch: 'master', url: 'https://github.com/jeeva113/gittest', credentialsId: 'github-creds'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} ."
                }
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                script {
                    withDockerRegistry([credentialsId: "${DOCKER_CRED_ID}", url: 'https://index.docker.io/v1/']) {
                        sh "docker tag ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                        sh "docker push ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                    }
                }
            }
        }

        stage('Update K8s Manifests') {
            steps {
                sh "sed -i 's#${REGISTRY}/${IMAGE_NAME}:.*#${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}#g' ${K8S_MANIFEST_PATH}/deployment.yaml"
            }
        }

        stage('Deploy to EKS') {
            steps {
                withAWS(region: "${AWS_REGION}", credentials: "${AWS_CRED_ID}") {
                    script {
                        echo '🚀 Updating kubeconfig...'
                        sh "aws eks update-kubeconfig --name myeks --region ${AWS_REGION} --kubeconfig ${WORKSPACE}/kubeconfig"

                        echo '🚀 Deploying to EKS...'
                        sh "${KUBECTL_PATH} --kubeconfig ${WORKSPACE}/kubeconfig apply -f ${K8S_MANIFEST_PATH}/deployment.yaml"
                        sh "${KUBECTL_PATH} --kubeconfig ${WORKSPACE}/kubeconfig apply -f ${K8S_MANIFEST_PATH}/service.yaml"
                        sh "${KUBECTL_PATH} --kubeconfig ${WORKSPACE}/kubeconfig rollout status deployment/calculator-deployment"
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment succeeded!'
        }
        failure {
            echo '❌ Deployment failed!'
        }
    }
}
