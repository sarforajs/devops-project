pipeline {
agent any

environment {
    IMAGE_NAME = "kubesarforaj/devops-app"
    TAG = "${BUILD_NUMBER}-${GIT_COMMIT[0..6]}"
}

stages {

    stage('Build Docker Image') {
        steps {
            sh 'docker build -t $IMAGE_NAME:$TAG ./app'
        }
    }

    stage('Security Scan') {
        steps {
            sh 'trivy image --exit-code 1 --severity HIGH,CRITICAL $IMAGE_NAME:$TAG'
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
