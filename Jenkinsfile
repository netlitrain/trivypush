pipeline {
    agent { label "${LABEL_NAME}" }

    environment {
        IMAGE_NAME = "trainerbpl10/webapp"
        IMAGE_TAG  = "${BUILD_NUMBER}"
        DOCKER_CREDS = credentials('dockerhub-creds')
    }

    stages {

        stage('Fetch Code') {
            steps {
                git url:"https://github.com/netlitrain/trivypush.git", branch:"main"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }

        stage('Scan Image with Trivy') {
            steps {
                sh '''
                echo "Scanning image with Trivy..."
                trivy image --exit-code 1 --severity HIGH,CRITICAL $IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }

        stage('Login to Docker Hub') {
            steps {
                sh '''
                echo "Logging into Docker Hub..."
                echo $DOCKER_CREDS_PSW | docker login -u $DOCKER_CREDS_USR --password-stdin
                '''
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                sh '''
                echo "Pushing image to Docker Hub..."
                docker push $IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                echo "Deploying container..."

                docker stop webapp || true
                docker rm webapp || true

                docker run -d \
                  --name webapp \
                  -p 80:80 \
                  $IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed. Check logs."
        }
    }
}
