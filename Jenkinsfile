pipeline {
    agent any
    environment {
        AWS_ACCOUNT_ID="318443738748"
        AWS_DEFAULT_REGION="us-east-2" 
        IMAGE_REPO_NAME="riktam-assign"
        IMAGE_TAG="${BUILD_NUMBER}"
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
    }

    stages {
        
         stage('Logging into AWS ECR') {
            steps {
                script {
                sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
                } 
            }
        }
        
        stage('Cloning Git') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '', url: 'https://github.com/aditya1431/RiktamAssignmentP.git']]])     
            }
        }
  
    // Building Docker images
    stage('Building image') {
      steps{
        script {
          dockerImage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}"
        }
      }
    }
   
    // Uploading Docker images into AWS ECR
    stage('Pushing to ECR') {
     steps{  
         script {
             
                sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:$IMAGE_TAG"
                sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
         }
        }
      }
    // Pulling Docker images into AWS ECR
    stage('Pull and deploy the image and run th container') {
        steps{
            script {
                withCredentials([sshUserPrivateKey(credentialsId: 'ssh-my-key', keyFileVariable: 'identity', passphraseVariable: '', usernameVariable: 'userName')]) {
                    def remote = [:]
                    remote.user = userName
                    remote.identityFile = identity
                    remote.name = 'webServer'
                    remote.host = '3.140.14.206'                
                    remote.allowAnyHosts = 'true'
                    sshCommand remote:remote, command: "uptime"
                    println "Stop all running containers"
                    sshCommand remote:remote, command: "docker stop \$(docker ps -a -q)"
                    println "Remove unused images and containers"
                    sshCommand remote:remote, command: "docker system prune -af"
                    println "pulling the image on remote host"
                    sshCommand remote:remote, command: "docker pull ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
                    println "Running the container now"
                    sshCommand remote:remote, command: "docker run -dp 80:3000 ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
            }
         }
        }
    }
    }
}
