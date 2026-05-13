pipeline {
  agent {
    label 'k8s-slave'
  }
  environment {
    REGISTRY_URL =  "docker.io"
    IMAGE_REPOSITORY = "dileepdocker5535/i27project-ui-microservice"
  }
  //prepareing the tag for the image to be built
  stages {
    stage ('prepare tag') {
      steps {
        script {
          env.IMAGE_NAME = "${env.REGISTRY_URL}/${env.IMAGE_REPOSITORY}"
          echo "Using Registry URL: ${env.REGISTRY_URL}"
          echo "Using Image Repository: ${env.IMAGE_REPOSITORY}"
          echo "Using Image Tag: ${GIT_COMMIT}"
          echo "Full Image Name: ${env.IMAGE_NAME}:${GIT_COMMIT}"
        }
        
      }
    }
  }
}
