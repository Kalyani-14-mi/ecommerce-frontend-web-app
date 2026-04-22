pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        DOCKERHUB_REPO        = "${DOCKERHUB_CREDENTIALS_USR}/ecommerce-frontend-web-app"
        IMAGE_TAG             = "${BUILD_NUMBER}"
    }
    stages {
        stage('Checkout') {
            steps {
                echo 'Cloning repository...'
                git branch: 'master',
                    url: 'https://github.com/Kalyani-14-mi/ecommerce-frontend-web-app.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                echo "Building Docker image: ${DOCKERHUB_REPO}:${IMAGE_TAG}"
                sh """
                    docker build -t ${DOCKERHUB_REPO}:${IMAGE_TAG} .
                    docker tag  ${DOCKERHUB_REPO}:${IMAGE_TAG} ${DOCKERHUB_REPO}:latest
                """
            }
        }
        stage('Push to DockerHub') {
            steps {
                echo 'Pushing image to DockerHub...'
                sh """
                    echo "${DOCKERHUB_CREDENTIALS_PSW}" | docker login -u "${DOCKERHUB_CREDENTIALS_USR}" --password-stdin
                    docker push ${DOCKERHUB_REPO}:${IMAGE_TAG}
                    docker push ${DOCKERHUB_REPO}:latest
                """
            }
        }
        stage('Cleanup') {
            steps {
                echo 'Removing local images to free disk space...'
                sh """
                    docker rmi ${DOCKERHUB_REPO}:${IMAGE_TAG} || true
                    docker rmi ${DOCKERHUB_REPO}:latest        || true
                    docker logout
                """
            }
        }
        stage('Deploy to K8s') {
            steps {
                sh 'kubectl apply -f k8s/namespace.yaml'
                sh 'kubectl wait --for=condition=Active namespace/ecommerce --timeout=30s'
                sh 'kubectl apply -f k8s/'
            }
        }
    }
    post {
        success {
            echo "✅ Image ${DOCKERHUB_REPO}:${IMAGE_TAG} pushed successfully."
        }
        failure {
            echo '❌ Pipeline failed. Check the logs above.'
        }
        always {
            cleanWs()
        }
    }
}
