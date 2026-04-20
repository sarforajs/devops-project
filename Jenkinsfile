pipeline {
    agent any

    environment {
        IMAGE_NAME = "kubesarforaj/devops-app"
        TAG = "v2"
    }

    stages {

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$TAG ./app'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sh 'docker push $IMAGE_NAME:$TAG'
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
}
