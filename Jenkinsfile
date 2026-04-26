pipeline {
    agent any

    environment {
        IMAGE_NAME = "kubesarforaj/devops-app"
        TAG = "${BUILD_NUMBER}-${GIT_COMMIT[0..6]}"
        TRIVY_CACHE_DIR = "/var/lib/jenkins/.cache/trivy"
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo "Checking out source code..."
                checkout scm
            }
        }

        stage('Build & Scan') {
            parallel {

                stage('Build Docker Image') {
                    steps {
                        echo "Building Docker image..."
                        sh '''
                        docker build -t $IMAGE_NAME:$TAG ./app
                        '''
                    }
                }

                stage('Security Scan (Trivy)') {
                    steps {
                        echo "Preparing Trivy cache..."
                        sh '''
                        mkdir -p $TRIVY_CACHE_DIR
                        sudo chown -R jenkins:jenkins $TRIVY_CACHE_DIR || true
                        '''

                        echo "Running Trivy scan..."
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
            }
        }

        stage('Push Image') {
            steps {
                echo "Pushing image to Docker Hub..."
                sh '''
                docker push $IMAGE_NAME:$TAG
                '''
            }
        }

        stage('Deploy to Kubernetes (Helm)') {
            steps {
                echo "Deploying via Helm..."
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
            echo "Cleaning workspace..."
            cleanWs()
        }

        success {
            echo "✅ Pipeline completed successfully!"
        }

        failure {
            echo "❌ Pipeline failed. Check logs."
        }
    }
}
