pipeline {
    agent { label 'master' }
    // agent any
    environment {
        AWS_ACCOUNT_ID="350515911022"
        AWS_DEFAULT_REGION="us-east-2"
        IMAGE_REPO_NAME="new_chatapp"
        IMAGE_TAG="""${BUILD_NUMBER}"""
        REPOSITORY_URL = "350515911022.dkr.ecr.us-east-2.amazonaws.com/new_chatapp"
    }
   
    stages {
	
        stage('git') {
            steps {
                // Get some code from a GitHub repository using SSH URL
                git credentialsId: 'GitHub', url: 'git@github.com:chaitanya1330/new_chatapp.git', branch: 'master'
            }
        }
        
        stage('Logging into AWS ECR') {
            steps {
                script {
                sh """aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"""

                }
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
                    sh """docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URL}:$IMAGE_TAG"""
                    sh """docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"""
                }
            }
        }
        
        // pulling the image from Amazon Elastic container registry
        stage('Pull Image from ECR') {
                steps {
                    script {
                        // Pull the Docker image from ECR
                        sh """docker pull ${REPOSITORY_URL}:${IMAGE_TAG}"""
                    }
                }
        }
        

        stage('Run Docker Container') {
                steps {
                    script {
                        // Run the Docker container
                        sh """docker rm -f app_cont"""
                        sh """docker run -d --name app_cont --network=docker_network_comp -p 8000:8000 ${REPOSITORY_URL}:${IMAGE_TAG}"""
                    }
                }
            }
            
            
    }
}
