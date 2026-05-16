pipeline {
  agent {
    label 'k8s-slave'
  }
  parameters {
      booleanParam(name: 'BUILD', defaultValue: true, description: "Run build and push image")
      choice(name: 'TRAGET_ENV', choices: ['dev', 'test', 'stage', 'prod'], description: "Target environement for API url")
      booleanParam(name: 'SKIP_SONAR', defaultValue: false, description: "Skip SonarQube analysis and Quality Gate")
  }
  environment {
    REGISTRY_URL =  "docker.io"
    IMAGE_REPOSITORY = "dileepdocker5535/i27-ui-microservice"
    REGISTRY_CREDENTIALS = credentials('docker_creds')
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
          switch(params.TRAGET_ENV) {
            case 'dev': env.NEXT_PUBLIC_API_BASE_URL = 'http://35.202.143.16:8080'
              break
            case 'test': env.NEXT_PUBLIC_API_BASE_URL = 'http://test-gateway.i27helpdesk.in'
              break
            case 'stage': env.NEXT_PUBLIC_API_BASE_URL = 'http://stage-gateway.i27helpdesk.in'
              break
            case 'prod': env.NEXT_PUBLIC_API_BASE_URL = 'http://gateway.i27helpdesk.in'
              break
          }
          echo "Using Registry URL: ${env.REGISTRY_URL}"
          echo "Using Image Repository: ${env.IMAGE_REPOSITORY}"
          echo "Using Image Tag: ${GIT_COMMIT}"
          echo "Full Image Name: ${env.IMAGE_NAME}:${GIT_COMMIT}"
        }
        
      }
    }
    stage ('SonarQube Analysis') {
      when {
        expression {
          return params.BUILD && !params.SKIP_SONAR
        }
      }
      steps {
         script {
          def scannerHome = tool 'SonarQubeScanner'
          withSonarQubeEnv('SonarQube') {
            sh "${scannerHome}/bin/sonar-scanner"
          }
        }
      }
    }
    stage ('Quality Gate') {
      when {
        expression {
          return params.BUILD && !params.SKIP_SONAR
        }
      }
      steps {
        timeout(time: 2, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
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
        sh "docker build -t ${env.IMAGE_NAME}:${GIT_COMMIT}  --build-arg  NEXT_PUBLIC_API_BASE_URL=${env.NEXT_PUBLIC_API_BASE_URL} ."
      }
    }
    stage ('push docker image') {
      when {
        expression {
          return params.BUILD
        }
      }
      steps {
        echo "************DOCKER LOGIN************"
        sh '''#!/bin/bash
            echo $REGISTRY_CREDENTIALS_PSW | docker login -u $REGISTRY_CREDENTIALS_USR --password-stdin $REGISTRY_URL
        '''
        echo "************DOCKER PUSH************"
        sh "docker push ${env.IMAGE_NAME}:${GIT_COMMIT}"
      }
    }
    //stage ('deploy to dev environment') {
      //Gke cluster should be available 
      //kubectl should be configured to access the cluster
      //slave should be having confige file to connect the cluster
    //}
  }
  post {
    always {
      cleanWs()
    }
    success {
      echo "Pipeline completed successfully!"
    }
    failure {
      echo "Pipeline failed. Please check the logs for details."
    }
  }
}

