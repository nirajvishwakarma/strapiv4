def targetK8SEnv = "dev"
def targetK8SCluster= "dev"

pipeline {


  agent any
    
  environment {
    registry = "276304551001.dkr.ecr.ap-south-1.amazonaws.com/strapiv4"
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
          sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 276304551001.dkr.ecr.ap-south-1.amazonaws.com'
          sh 'docker push 276304551001.dkr.ecr.ap-south-1.amazonaws.com/strapiv4:latest'
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
          sh 'docker run -d -p 1337:1337 --rm --name strapi_container 276304551001.dkr.ecr.ap-south-1.amazonaws.com/strapiv4:latest'
        }
      }
    }
   
    stage('Interactive') {

      agent any

      steps {
        script {
          timeout ( time: 2, unit: "HOURS" ) {
            userAns = input(
              message: "Where to deploy to?",
              parameters: [choice(choices: ['dev', 'sit', 'demo', 'prod'],
              description: 'k8s env',
              name: 'deployTo')]
            )
          }

          switch(userAns) {
            case ["dev", "sit"]:
              targetK8SCluster = "dev"
              break
            case ["demo", "prod"]:
              targetK8SCluster = "prod"
              break
            default:
              error("Unknown error")
          }

          targetK8SEnv = userAns

          currentBuild.displayName = "${env.BUILD_ID}-${targetK8SEnv}-${env.TAG_NAME}"
        }
      }
    }
    
    
    stage ('K8S Deploy') {
        parallel {
                stage('deploy to development') {
                    when {
                      expression { targetK8SCluster == "dev" }
                    }
                    steps {
                        sh 'aws eks --region us-east-1 update-kubeconfig --name WIM-dev'
                        kubernetesDeploy(configs: "pg-deployment.yaml", kubeconfigId: "WIM-dev")
                        kubernetesDeploy(configs: "strapi-deployment.yaml", kubeconfigId: "WIM-dev")
                    }
                }
                stage('deploy to production') {
                    when {
                        expression { targetK8SCluster == "prod" }
                    }
                    steps {
                        sh 'aws eks --region us-east-1 update-kubeconfig --name WIM-prod'
           	            kubernetesDeploy(configs: "pg-deployment.yaml", kubeconfigId: "WIM-prod")
                        kubernetesDeploy(configs: "strapi-deployment.yaml", kubeconfigId: "WIM-prod")
                    }
                }
            }
    }
    
    
    
    stage("Completed") {
        steps {
            echo "Deployment Completed to    the ${targetK8SCluster} environment."
        }
    }
   
   
   
  }
}
