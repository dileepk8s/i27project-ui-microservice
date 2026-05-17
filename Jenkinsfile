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
    // kubernetes Dev Cluser Details
    DEV_CLUSTER_NAME = "np-cluster"
    DEV_CLUSTER_ZONE = "us-east4-a"
    DEV_CLUSTER_PROJECT = "dileepk8s-495003"
     // kubernetes Test Cluser Details
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
    stage ('GKE AUTH') {
      when {
        expression {
          return params.TRAGET_ENV == 'dev'
        }
      }
      steps {
        script {
          sh """
          echo "*****************Authenticating to GKE Cluster*****************************"
          gcloud container clusters get-credentials ${env.DEV_CLUSTER_NAME} --zone ${env.DEV_CLUSTER_ZONE} --project ${env.DEV_CLUSTER_PROJECT}
          echo "*****************Validating the cluster access*****************************"
          kubectl get nodes
          """
        }
      }
    }
    stage ('Deploy To Dev Environment') {
      //Gke cluster should be available
      //kubectl should be configured to access the cluster
      //slave should be having confige file to connect the cluster
      //create k8s manifest file and make them apply into our namesapce 
      //create reuseable code for all environment deployment and use the same code for all environment with different parameters
      when {
        expression {
          return params.BUILD && params.TRAGET_ENV == 'dev'
        }
      }
      steps {
        script {
          env.NAMESPACE = "i27-helpdesk-dev"
          sh '''
          echo "*****************Deploying to Dev Environment*****************************"
          echo "Deploying in the namespace: ${NAMESPACE}"
          kubectl get  pods -n ${NAMESPACE}
          #substitute variables in the manifest files
          sed -i "s|\\${NAMESPACE}|${NAMESPACE}|g" k8s/*yaml
          sed -i "s|\\${IMAGE_NAME}|${IMAGE_NAME}|g" k8s/deploy.yaml
          sed -i "s|\\${IMAGE_TAG}|${GIT_COMMIT}|g" k8s/deploy.yaml
          echo "Applying Kubernetes manifests in dev namespace"
          kubectl apply -f k8s/
          echo "Deployment to Dev Environment completed successfully!"
          '''
        }
      }
    }
    stage ('Deploy To Test Environment') {
      //Gke cluster should be available
      //kubectl should be configured to access the cluster
      //slave should be having confige file to connect the cluster
      //create k8s manifest file and make them apply into our namesapce 
      //create reuseable code for all environment deployment and use the same code for all environment with different parameters
      when {
        expression {
          return params.BUILD && params.TRAGET_ENV == 'test'
        }
      }
      steps {
        script {
          env.NAMESPACE = "i27-helpdesk-test"
          sh '''
          echo "*****************Deploying to Test Environment*****************************"
          echo "Deploying in the namespace: ${NAMESPACE}"
          kubectl get  pods -n ${NAMESPACE}
          #substitute variables in the manifest files
          sed -i "s|\\${NAMESPACE}|${NAMESPACE}|g" k8s/*yaml
          sed -i "s|\\${IMAGE_NAME}|${IMAGE_NAME}|g" k8s/deploy.yaml
          sed -i "s|\\${IMAGE_TAG}|${GIT_COMMIT}|g" k8s/deploy.yaml
          echo "Applying Kubernetes manifests in test namespace"
          kubectl apply -f k8s/
          echo "Deployment to Test Environment completed successfully!"
          '''
        }
      }
    }
    stage ('Deploy To Stage Environment') {
      //Gke cluster should be available
      //kubectl should be configured to access the cluster
      //slave should be having confige file to connect the cluster
      //create k8s manifest file and make them apply into our namesapce 
      //create reuseable code for all environment deployment and use the same code for all environment with different parameters
      when {
        expression {
          return params.BUILD && params.TRAGET_ENV == 'stage'
        }
      }
      steps {
        script {
          env.NAMESPACE = "i27-helpdesk-stage"
          sh '''
          echo "*****************Deploying to Stage Environment*****************************"
          echo "Deploying in the namespace: ${NAMESPACE}"
          kubectl get  pods -n ${NAMESPACE}
          #substitute variables in the manifest files
          sed -i "s|\\${NAMESPACE}|${NAMESPACE}|g" k8s/*yaml
          sed -i "s|\\${IMAGE_NAME}|${IMAGE_NAME}|g" k8s/deploy.yaml
          sed -i "s|\\${IMAGE_TAG}|${GIT_COMMIT}|g" k8s/deploy.yaml
          echo "Applying Kubernetes manifests in stage namespace"
          kubectl apply -f k8s/
          echo "Deployment to Stage Environment completed successfully!"
          '''
        }
      }
    }
    stage ('Deploy To Prod Environment') {
      //Gke cluster should be available
      //kubectl should be configured to access the cluster
      //slave should be having confige file to connect the cluster
      //create k8s manifest file and make them apply into our namesapce 
      //create reuseable code for all environment deployment and use the same code for all environment with different parameters
      when {
        expression {
          return params.BUILD && params.TRAGET_ENV == 'prod'
        }
      }
      steps {
        script {
          env.NAMESPACE = "i27-helpdesk-prod"
          sh '''
          echo "*****************Deploying to Prod Environment*****************************"
          echo "Deploying in the namespace: ${NAMESPACE}"
          kubectl get  pods -n ${NAMESPACE}
          #substitute variables in the manifest files
          sed -i "s|\\${NAMESPACE}|${NAMESPACE}|g" k8s/*yaml
          sed -i "s|\\${IMAGE_NAME}|${IMAGE_NAME}|g" k8s/deploy.yaml
          sed -i "s|\\${IMAGE_TAG}|${GIT_COMMIT}|g" k8s/deploy.yaml
          echo "Applying Kubernetes manifests in prod namespace"
          kubectl apply -f k8s/
          echo "Deployment to Prod Environment completed successfully!"
          '''
        }
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
      echo "Pipeline failed. Please check the logs for details."
    }
  }
}

