pipeline {
    agent any

    environment {
        registry = "119741951591.dkr.ecr.us-east-1.amazonaws.com/jenkins-ecr"

    }
    stages {
        stage('Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/srinivasulureddy22/azurejenkins']])
            }
        }
        
        stage ("Build JAR") {
            steps {
                sh "mvn clean install"
            }
        }
        
    // Building Docker images
        stage('Building image') {
         steps{
          script {
           dockerImage = docker.build registry 
           dockerImage.tag("$BUILD_NUMBER")
          }
         }
      }
        
      stage ("Push to ECR") {
          steps {
                script {
                    sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 119741951591.dkr.ecr.us-east-1.amazonaws.com"
                    sh "docker push 119741951591.dkr.ecr.us-east-1.amazonaws.com/jenkins-ecr:latest"
                    
                }
            }
        }
    stage('k8s deployment') {
        steps {
            sh 'kubectl apply -f eks-deploy.yaml'
        }
    }  
        
   //     stage ("Helm package") {
   //         steps {
   //                 sh "helm package springboot"
   //             }
   //         }
                
   //     stage ("Helm install") {
   //         steps {
   //                 sh "helm upgrade myrelease-21 springboot-0.1.0.tgz"
   //             }
   //         }
    }
}
