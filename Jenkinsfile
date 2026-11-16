pipeline {
  agent any
  environment {
    IMAGE = "cicd-demo"
    CONTAINER = "cicd-demo-app"
  }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Test') {
      steps {
        sh '''
          docker run --rm -u $(id -u):$(id -g) -e HOME=/tmp \
            -v "$PWD":/app -w /app node:20-alpine \
            sh -c "npm ci && npm test"
        '''
      }
    }
    stage('Docker Build') {
      steps {
        sh 'docker build -t $IMAGE:$BUILD_NUMBER -t $IMAGE:latest .'
      }
    }
    stage('Deploy') {
      steps {
        sh '''
          docker stop $CONTAINER || true
          docker rm $CONTAINER || true
          docker run -d --name $CONTAINER --restart unless-stopped -p 3000:3000 $IMAGE:$BUILD_NUMBER
          sleep 5
          curl -f http://localhost:3000/health
        '''
      }
    }
  }
  post {
    failure { echo 'Pipeline failed, check the logs above' }
    always { sh 'docker image prune -f' }
  }
}

