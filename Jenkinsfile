pipeline {
    agent any

    environment {
        REGISTRY = 'jeeva1306'
        IMAGE_NAME = 'myapp'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        DOCKER_CRED_ID = 'dockerhub-creds'
        K8S_MANIFEST_PATH = "${WORKSPACE}/k8s"  // full path to k8s folder
        AWS_REGION = 'us-east-1'
        EKS_CLUSTER = 'myeks'
        KUBECTL_PATH = '/root/bin/kubectl'      // full path to kubectl
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'master', url: 'https://github.com/jeeva113/gittest.git', credentialsId: 'github-creds'
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
                sh """
                    sed -i "s#${REGISTRY}/${IMAGE_NAME}:.*#${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}#g" ${K8S_MANIFEST_PATH}/deployment.yaml
                """
            }
        }

        stage('Deploy to EKS') {
            steps {
                script {
                    echo '🚀 Updating kubeconfig...'
                    sh "aws eks update-kubeconfig --name ${EKS_CLUSTER} --region ${AWS_REGION} --kubeconfig ${WORKSPACE}/kubeconfig"

                    echo '🚀 Deploying to EKS...'
                    sh "${KUBECTL_PATH} --kubeconfig ${WORKSPACE}/kubeconfig apply -f ${K8S_MANIFEST_PATH}/deployment.yaml"
                    sh "${KUBECTL_PATH} --kubeconfig ${WORKSPACE}/kubeconfig apply -f ${K8S_MANIFEST_PATH}/service.yaml"
                    sh "${KUBECTL_PATH} --kubeconfig ${WORKSPACE}/kubeconfig rollout status deployment/calculator-deployment"
                }
            }
        }
    }

    post {
        success {
            echo "✅ App deployed successfully! Check ELB via '${KUBECTL_PATH} --kubeconfig ${WORKSPACE}/kubeconfig get svc calculator-service'"
        }
        failure {
            echo "❌ Deployment failed!"
        }
    }
}
