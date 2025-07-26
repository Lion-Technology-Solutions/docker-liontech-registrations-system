pipeline {
    agent any
    
    environment {
        AWS_ACCOUNT_ID = '768477844960'
        AWS_REGION = 'ca-central-1'
        ECR_REPO = 'demo-app'
        DOCKER_IMAGE = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}"
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        EC2_INSTANCE_IP = '35.183.70.86'
        SSH_USER = 'ubuntu' // or ubuntu depending on AMI
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/Lion-Technology-Solutions/docker-liontech-registrations-system.git'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                }
            }
        }
        
        stage('Login to ECR') {
            steps {
                script {
                    sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
                }
            }
        }
        
        stage('Push to ECR') {
            steps {
                script {
                    docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                    // Optionally push as 'latest'
                    //docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push('latest')
                }
            }
        }
        
        stage('Deploy to EC2') {
            steps {
                script {
                    // SSH into EC2 and deploy the new image
                    withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh-key', keyFileVariable: 'SSH_KEY')]) {
                        sh """
                            ssh -o StrictHostKeyChecking=no -i ${SSH_KEY} ${SSH_USER}@${EC2_INSTANCE_IP} << EOF
                            # Pull the new image
                            aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                            docker pull ${DOCKER_IMAGE}:${DOCKER_TAG}
                            
                            # Stop and remove old container
                            docker stop your-app-name || true
                            docker rm your-app-name || true
                            
                            # Run new container
                            docker run -d --name your-app-name -p 80:8080 ${DOCKER_IMAGE}:${DOCKER_TAG}
                            # Add any other run options needed
                            EOF
                        """
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}