pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        AWS_ACCOUNT_ID = '<YOUR_AWS_ACCOUNT_ID>'
        IMAGE_REPO_NAME = 'webapp-ci-cd-demo'
        IMAGE_TAG = "v${env.BUILD_NUMBER}"
        ECR_URL = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/<your-username>/webapp-ci-cd-demo.git'
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
                withAWS(credentials: 'aws-jenkins', region: "${AWS_REGION}") {
                    script {
                        sh """
                        aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                        """
                    }
                }
            }
        }

        stage('Push Image to ECR') {
            steps {
                script {
                    docker.image("${IMAGE_REPO_NAME}:${IMAGE_TAG}").tag("${ECR_URL}:${IMAGE_TAG}")
                    sh "docker push ${ECR_URL}:${IMAGE_TAG}"
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                echo 'Deploying container on EC2...'
                // Example: SSH into EC2 and run docker pull/run commands
                sshagent (credentials: ['ec2-ssh-key']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ec2-user@<EC2_PUBLIC_IP> \
                    'docker pull ${ECR_URL}:${IMAGE_TAG} && \
                    docker stop webapp || true && docker rm webapp || true && \
                    docker run -d -p 80:3000 --name webapp ${ECR_URL}:${IMAGE_TAG}'
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
                to: 'your-email@example.com'
            )
        }
        failure {
            emailext (
                subject: "FAILURE: Build #${BUILD_NUMBER}",
                body: "The pipeline failed. Please check the Jenkins logs.",
                to: 'your-email@example.com'
            )
        }
    }
}