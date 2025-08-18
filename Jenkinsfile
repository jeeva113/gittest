pipeline {
    agent any

    environment {
        REGISTRY = 'jeeva1306'
        IMAGE_NAME = 'myapp'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        DOCKER_CRED_ID = 'dockerhub-creds'
        K8S_MANIFEST_PATH = 'k8s'  // folder where deployment.yaml & service.yaml are kept
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
                sh '''
                  echo "🚀 Deploying to EKS..."
                  kubectl apply -f ${K8S_MANIFEST_PATH}/deployment.yaml
                  kubectl apply -f ${K8S_MANIFEST_PATH}/service.yaml
                  kubectl rollout status deployment/calculator-deployment
                '''
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
}
