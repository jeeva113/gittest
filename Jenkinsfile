pipeline {
    agent any

    environment {
        REGISTRY = 'jeeva1306'
        IMAGE_NAME = 'myapp'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        DOCKER_CRED_ID = 'dockerhub-creds'
        K8S_MANIFEST_PATH = 'k8s'
        KUBECTL_PATH = '/usr/local/bin/kubectl' // updated path
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
                script {
                    withDockerRegistry([credentialsId: "${DOCKER_CRED_ID}", url: "https://index.docker.io/v1/"]) {
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
                withAWS(credentials: 'aws-eks-creds', region: 'us-east-1') {
                    echo '🚀 Updating kubeconfig...'
                    sh "aws eks update-kubeconfig --name myeks --region us-east-1 --kubeconfig \$WORKSPACE/kubeconfig"

                    echo '🚀 Deploying to EKS...'
                    sh "${KUBECTL_PATH} --kubeconfig \$WORKSPACE/kubeconfig apply -f \$WORKSPACE/${K8S_MANIFEST_PATH}/deployment.yaml"
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
