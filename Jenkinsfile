pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        AWS_ACCOUNT_ID = '972775291931'
        IMAGE_REPO_NAME = 'cicd'
        IMAGE_TAG = "v${env.BUILD_NUMBER}"
        ECR_URL = "972775291931.dkr.ecr.us-east-1.amazonws.com/cicd"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Sanjana-301/cicd.git'
            }
        }

        stage('Build Application') {
            steps {
                echo 'Installing dependencies...'
                bat 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                echo 'Running unit tests...'
                bat 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${IMAGE_REPO_NAME}:${IMAGE_TAG}")
                }
            }
        }

        stage('Login to AWS ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-jenkins']]) {
                    bat """
                    aws ecr get-login-password --region ${AWS_REGION} ^
                    | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                    """
                }
            }
        }

        stage('Push Image to ECR') {
            steps {
                bat """
                docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}
                docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Deploy to EC2') {
            steps {
                echo 'Deploying container on EC2...'
                withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh-key', keyFileVariable: 'SSH_KEY')]) {
    bat """
    ssh -i "%SSH_KEY%" -o StrictHostKeyChecking=no ec2-user@52.23.185.236 ^
    "docker pull ${ECR_URL}:${IMAGE_TAG} && ^
    docker stop webapp || exit /b 0 && docker rm webapp || exit /b 0 && ^
    docker run -d -p 80:3000 --name webapp ${ECR_URL}:${IMAGE_TAG}"
    """
}
            }
        }
    }

    post {
        success {
            emailext (
                subject: "SUCCESS: Build #${BUILD_NUMBER}",
                body: "The pipeline has been successfully executed and deployed.",
                to: 'sanjana4047@example.com'
            )
        }
        failure {
            emailext (
                subject: "FAILURE: Build #${BUILD_NUMBER}",
                body: "The pipeline failed. Please check the Jenkins logs.",
                to: 'sanjana4047@example.com'
            )
        }
    }
}
