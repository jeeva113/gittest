pipeline {
    agent any

    environment {
        REGISTRY = 'jeeva1306'             // your DockerHub username
        IMAGE_NAME = 'myapp'
        IMAGE_TAG = "${env.BUILD_NUMBER}"  // auto tag with build number
        DOCKER_CRED_ID = 'dockerhub-creds' // Jenkins Docker Hub credentials ID
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

        stage('Run Tests') {
            steps {
                sh '''
                    echo "✅ Running tests..."
                    node -v    # verify Node.js installed inside Jenkins node
                '''
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

        stage('Run Container') {
            steps {
                sh '''
                    echo "🚀 Running container..."
                    docker rm -f calculator-app || true
                    docker run -d --name calculator-app -p 5000:5000 ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    echo "🔍 Checking if app is responding..."
                    sleep 5
                    curl -f http://localhost:5000 || (echo "❌ App did not start correctly!" && exit 1)
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Build & Deployment Successful! Access app at http://<EC2-PUBLIC-IP>:5000"
        }
        failure {
            echo "❌ Build or Deployment Failed!"
        }
    }
}
