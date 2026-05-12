pipeline{
  agent {
    label 'k8s-slave'
  }
  environment {
    REGISTRY_URL =  "docker.io"
    IMAGE_REPOSITORY = "dileepk8s/i27project-ui-microservice"
  }
  //prepareing the tag for the image to be built
  stages {
    stage ('prepare tag') {
      steps {
        echo "Using Registry URL: ${env.REGISTRY_URL}"
        echo "Using Image Repository: ${env.IMAGE_REPOSITORY}"
        echo "Using Image Tag: ${GIT_COMMIT}"
      }
    }
    //stage ('Build DOcker Image')
  }
}