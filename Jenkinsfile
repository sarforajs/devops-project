pipeline {
    agent any

    environment {
        IMAGE_NAME = "kubesarforaj/devops-app"
        TAG = "${BUILD_NUMBER}-${GIT_COMMIT[0..6]}"
        TRIVY_CACHE_DIR = "/var/lib/jenkins/.cache/trivy"
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

        stage('Prepare Trivy Cache') {
            steps {
                sh '''
                mkdir -p $TRIVY_CACHE_DIR
                sudo chown -R jenkins:jenkins $TRIVY_CACHE_DIR || true
                '''
            }
        }

        stage('Security Scan') {
            steps {
                sh '''
                docker run --rm \
                  -v /var/run/docker.sock:/var/run/docker.sock \
                  -v $TRIVY_CACHE_DIR:/root/.cache/trivy \
                  aquasec/trivy image \
                  --scanners vuln \
                  --timeout 15m \
                  --exit-code 1 \
                  --severity HIGH,CRITICAL \
                  --cache-dir /root/.cache/trivy \
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
            echo "Pipeline failed. Rolling back Helm release..."
            sh 'helm rollback myapp || true'
        }
    }
}
