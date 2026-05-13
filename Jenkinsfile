pipeline {
  agent {
    label 'k8s-slave'
  }
  parameters {
    booleanParameter(name: 'BUILD', defaultValue: true, description: "Run build and push image")
  }
  environment {
    REGISTRY_URL =  "docker.io"
    IMAGE_REPOSITORY = "dileepdocker5535/i27project-ui-microservice"
  }
  //prepareing the tag for the image to be built
  stages {
    stage ('prepare tag') { 
      when {
        expression {
          return params.BUILD
        }
      }
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
    stage ('build docker image') {
      when {
        expression {
          return params.BUILD
        }
      }
      steps {
        echo "Building the image"
        //sh "docker build -t ${env.IMAGE_NAME}:${GIT_COMMIT}  --build-arg  NEXT_PUBLIC_API_BASE_URL=http://35.202.143.16:8080 ."
      }
    }
  }
}
