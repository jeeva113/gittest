pipeline {
    agent any

    environment {
        REGISTRY = 'jeeva1306'
        IMAGE_NAME = 'myapp'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        DOCKER_CRED_ID = 'dockerhub-creds'
        K8S_MANIFEST_PATH = 'k8s'       // relative to repo root
        AWS_CRED_ID = 'aws-eks-creds'
        AWS_REGION = 'us-east-1'
        EKS_CLUSTER = 'myeks'
        KUBECONFIG_PATH = "${WORKSPACE}/kubeconfig"
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
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CRED_ID}") {
                        docker.image("${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}").push()
                    }
                }
            }
        }

        stage('Update K8s Manifests') {
            steps {
                sh """
                    sed -i 's#${REGISTRY}/${IMAGE_NAME}:.*#${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}#g' ${K8S_MANIFEST_PATH}/deployment.yaml
                """
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${AWS_CRED_ID}"]]) {
                    sh """
                        echo '🚀 Updating kubeconfig...'
                        aws eks update-kubeconfig --name ${EKS_CLUSTER} --region ${AWS_REGION} --kubeconfig ${KUBECONFIG_PATH}

                        echo '🚀 Deploying to EKS...'
                        kubectl --kubeconfig ${KUBECONFIG_PATH} apply -f ${WORKSPACE}/${K8S_MANIFEST_PATH}/deployment.yaml
                        kubectl --kubeconfig ${KUBECONFIG_PATH} apply -f ${WORKSPACE}/${K8S_MANIFEST_PATH}/service.yaml

                        echo '🚀 Waiting for rollout to complete...'
                        kubectl --kubeconfig ${KUBECONFIG_PATH} rollout status deployment/calculator-deployment
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful! Check ELB via: kubectl --kubeconfig ${KUBECONFIG_PATH} get svc calculator-service"
        }
        failure {
            echo "❌ Deployment failed!"
        }
    }
}
