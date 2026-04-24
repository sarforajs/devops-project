pipeline {
    agent any

    environment {
        IMAGE_NAME = "kubesarforaj/devops-app"
        TAG = "${BUILD_NUMBER}-${GIT_COMMIT[0..6]}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $IMAGE_NAME:$TAG ./app
                '''
            }
        }

        stage('Security Scan') {
            steps {
                sh '''
                docker run --rm \
                  -v /var/run/docker.sock:/var/run/docker.sock \
                  -v /var/lib/jenkins/.cache/trivy:/root/.cache/ \
                  aquasec/trivy image \
                  --timeout 10m \
                  --exit-code 1 \
                  --severity HIGH,CRITICAL \
                  $IMAGE_NAME:$TAG
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sh '''
                docker push $IMAGE_NAME:$TAG
                '''
            }
        }

        stage('Deploy using Helm') {
            steps {
                sh '''
                helm upgrade --install myapp ./mychart \
                  --set image.repository=$IMAGE_NAME \
                  --set image.tag=$TAG
                '''
            }
        }
    }

    post {
        always {
            cleanWs()
        }

        success {
            echo "Pipeline completed successfully!"
        }

        failure {
            echo "Pipeline failed. Check logs for details."
        }
    }
}
