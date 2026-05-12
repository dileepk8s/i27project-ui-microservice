pipeline {
  agent {
    label 'k8s-slave'
  }
  environment {
    REGISTRY_URL = "docker.io"
    IMAGE_REPOSITORY = "dileepk8s/i27project-ui-microservice"
  }
  stages {
    stage('Prepare tag') {
      steps {
        script {
          env.IMAGE_TAG = env.GIT_COMMIT ?: 'latest'
          env.IMAGE_NAME = "${env.REGISTRY_URL}/${env.IMAGE_REPOSITORY}"
          echo "Using Registry URL: ${env.REGISTRY_URL}"
          echo "Using Image Repository: ${env.IMAGE_REPOSITORY}"
          echo "Using Image Tag: ${env.IMAGE_TAG}"
          echo "Full Image Name: ${env.IMAGE_NAME}:${env.IMAGE_TAG}"
        }
      }
    }
    stage('Build Docker Image') {
      steps {
        script {
          def fullImageName = "${env.IMAGE_NAME}:${env.IMAGE_TAG}"
          docker.build(fullImageName)
        }
      }
    }
    stage('Push Docker Image') {
      steps {
        script {
          def fullImageName = "${env.IMAGE_NAME}:${env.IMAGE_TAG}"
          docker.withRegistry("https://${env.REGISTRY_URL}") {
            docker.image(fullImageName).push()
          }
        }
      }
    }
  }
}