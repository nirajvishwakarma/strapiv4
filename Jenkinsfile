def targetK8SEnv = "dev"
def targetK8SCluster= "dev"

pipeline {


  agent any
    
  environment {
    registry = "817141239014.dkr.ecr.us-east-1.amazonaws.com/strapiv4"
  }
  
  stages {

    stage ("checkout") {
      steps {
        checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/nirajvishwakarma/strapiv4.git']]])
       }
    }
   
    stage ("Docker build") {
      steps {
        script {
          dockerImage = docker.build registry
        }
       }
    }
        
    stage ("Docker upload") {
      steps {
        script {
          sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 817141239014.dkr.ecr.us-east-1.amazonaws.com'
          sh 'docker push 817141239014.dkr.ecr.us-east-1.amazonaws.com/strapiv4:latest'
        }
      }
            
    }
    
    stage ("Stop previos container"){
      steps {
        sh 'docker ps -f name=strapi_container -q | xargs --no-run-if-empty docker container stop'
        sh 'docker container ls -a -fname=strapi_container -q | xargs -r docker container rm'
      }
    }
        
    stage ("Docker run") {
      steps {
        script {
          sh 'docker run -d -p 1337:1337 --rm --name strapi_container 817141239014.dkr.ecr.us-east-1.amazonaws.com/strapiv4:latest'
        }
      }
    }
   
    stage("Completed") {
        steps {
            echo "Deployment Completed to the ${targetK8SCluster} environment."
        }
    }
   
   
   
  }
}
