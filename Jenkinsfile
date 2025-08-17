pipeline {
    agent any

    environment {
        REGISTRY = 'jeeva1306'   // change to your DockerHub username
        IMAGE_NAME = 'myapp'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        DOCKER_CRED_ID = 'dockerhub-creds'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/jeeva113/gittest.git'
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
                    node -v    # check Node installed in image
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
