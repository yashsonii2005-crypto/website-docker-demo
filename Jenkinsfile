pipeline {
    agent any
    
    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPOSITORY = '317707834474.dkr.ecr.ap-south-1.amazonaws.com/website-docker-demo'
        IMAGE_TAG = '${BUILD_NUMBER}'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/yashsonii2005-crypto/website-docker-demo.git'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t website-docker-demo .'
                }
            }
        }
        
        stage('Tag Docker Image') {
            steps {
                script {
                    sh "docker tag website-docker-demo:latest ${ECR_REPOSITORY}:${IMAGE_TAG}"
                    sh "docker tag website-docker-demo:latest ${ECR_REPOSITORY}:latest"
                }
            }
        }
        
        stage('Login to ECR') {
            steps {
                script {
                    sh 'aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 317707834474.dkr.ecr.ap-south-1.amazonaws.com'
                }
            }
        }
        
        stage('Push Image to ECR') {
            steps {
                script {
                    sh "docker push ${ECR_REPOSITORY}:${IMAGE_TAG}"
                    sh "docker push ${ECR_REPOSITORY}:latest"
                }
            }
        }
        
        stage('Deploy to EC2') {
            steps {
                script {
                    sh '''
                    ssh -o StrictHostKeyChecking=no -i /var/lib/jenkins/deploy-ec2-key.pem ec2-user@65.0.79.239 '
                        docker stop website || true
                        docker rm website || true
                        aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 317707834474.dkr.ecr.ap-south-1.amazonaws.com
                        docker pull 317707834474.dkr.ecr.ap-south-1.amazonaws.com/website-docker-demo:latest
                        docker run -d --name website -p 80:80 317707834474.dkr.ecr.ap-south-1.amazonaws.com/website-docker-demo:latest
                    '
                    '''
                }
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline completed'
        }
        success {
            echo 'Pipeline succeeded'
        }
        failure {
            echo 'Pipeline failed'
        }
    }
}
