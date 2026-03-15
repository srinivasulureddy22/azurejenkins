pipeline {
    agent any

    parameters {
        choice(name: 'ENVIRONMENT', choices: ['dev', 'test'], description: 'Select environment to deploy')
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Docker image tag')
    }

    environment {
        AWS_REGION = "ap-south-1"
        ACCOUNT_ID = "119741951591"
        IMAGE_NAME = "jenkins-ecr"
        REGISTRY = "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${IMAGE_NAME}"
        DEV_CLUSTER = "dev-eks-cluster"
        TEST_CLUSTER = "test-eks-cluster"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/srinivasulureddy22/azurejenkins'
            }
        }

        stage('Build JAR') {
            steps {
                sh 'mvn clean install'
            }
        }

//        stage('SonarQube Scan') {
//            steps {
//                withSonarQubeEnv('sonar-server') {
//                    sh 'mvn sonar:sonar'
//                }
//            }
//        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${REGISTRY}:${params.IMAGE_TAG} ."
            }
        }

 //       stage('Trivy Scan') {
 //           steps {
 //              sh "trivy image ${REGISTRY}:${params.IMAGE_TAG}"
 //           }
 //       }

        stage('Push Image to ECR') {
            steps {
                sh """
                aws ecr get-login-password --region ${AWS_REGION} | \
                docker login --username AWS --password-stdin ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com

                docker push ${REGISTRY}:${params.IMAGE_TAG}
                """
            }
        }

        stage('Deploy to EKS') {
            steps {
                script {

                    if (params.ENVIRONMENT == "dev") {
                        sh """
                        aws eks update-kubeconfig --region ${AWS_REGION} --name ${DEV_CLUSTER}
                        kubectl apply -f k8s/dev-deploy.yaml
                        """
                    }

                    if (params.ENVIRONMENT == "test") {
                        sh """
                        aws eks update-kubeconfig --region ${AWS_REGION} --name ${TEST_CLUSTER}
                        kubectl apply -f k8s/test-deploy.yaml
                        """
                    }
                }
            }
        }
    }
}
