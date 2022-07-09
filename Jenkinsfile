pipeline {
  agent any
    tools {
      maven 'maven-jenkins'
      jdk 'jdk-11-jenkins'
    }
    environment{
        AWS_ACCOUNT_ID="466243422136"
        AWS_DEFAULT_REGION="ap-south-1" 
        IMAGE_REPO_NAME="demo06072022"
        IMAGE_TAG="latest"
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"	
        EMAIL_TO = 'kalleshdevops@gmail.com'		
    }
    stages{
        stage('Logging into AWS ECR') {
            steps {
                script {
                sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
                }
             }
        }	         
        stage('Build maven') {
            steps { 
                    sh 'pwd'      
                    sh 'mvn  clean install package'
	          }   
            }
        
        stage('Copy Artifact') {
           steps { 
                   sh 'pwd'
		   sh 'cp -r target/*.jar docker'
           }
        }
        stage('Building image') {
           steps{
           script {
           dockerImage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}","./docker"
        }
      }
    }
    	stage('Pushing to ECR') {
           steps{  
            script {
                sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:$IMAGE_TAG"
                sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
         }
        }
      }
	    stage('Deploy App on k8s') {
           steps {
            sshagent(['ssh-k8s-master']) {
            sh "scp -o StrictHostKeyChecking=no java-app-deployment.yaml ec2-user@3.108.64.225:/home/ec2-user"
            script {
                try{
                  sh "ssh ec2-user@3.108.64.225 kubectl create -f java-app-deployment.yaml"
                }catch(error){
                    sh "ssh ec2-user@3.108.64.225 kubectl apply -f java-app-deployment.yaml"
            }
}
        }
    }
}  
    }
}

