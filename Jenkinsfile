pipeline {
    agent any

    environment {
        REGISTRY = 'jeeva1306'
        IMAGE_NAME = 'myapp'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        DOCKER_CRED_ID = 'dockerhub-creds'
        K8S_MANIFEST_PATH = "${WORKSPACE}/k8s"
        KUBECTL_PATH = '/root/bin/kubectl'
        AWS_REGION = 'us-east-1'
        EKS_CLUSTER_NAME = 'myeks'
        AWS_CRED_ID = 'aws-eks-creds'
    }

    stages {
        stage('Checkout Code') {
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
                withDockerRegistry([credentialsId: "${DOCKER_CRED_ID}", url: "https://index.docker.io/v1/"]) {
                    sh "docker tag ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker push ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
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
                withAWS(credentials: "${AWS_CRED_ID}", region: "${AWS_REGION}") {
                    echo "🚀 Updating kubeconfig..."
                    sh "aws eks update-kubeconfig --name ${EKS_CLUSTER_NAME} --region ${AWS_REGION} --kubeconfig ${WORKSPACE}/kubeconfig"

                    echo "🚀 Deploying to EKS..."
                    sh "${KUBECTL_PATH} --kubeconfig ${WORKSPACE}/kubeconfig apply -f ${K8S_MANIFEST_PATH}/deployment.yaml"
                    sh "${KUBECTL_PATH} --kubeconfig ${WORKSPACE}/kubeconfig apply -f ${K8S_MANIFEST_PATH}/service.yaml"
                    sh "${KUBECTL_PATH} --kubeconfig ${WORKSPACE}/kubeconfig rollout status deployment/calculator-deployment"
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment succeeded!"
        }
        failure {
            echo "❌ Deployment failed!"
        }
    }
}
