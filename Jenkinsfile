pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_URL = '187038415085.dkr.ecr.ap-south-1.amazonaws.com'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        // 🔍 DEBUG (REMOVE LATER)
        stage('Debug Files') {
            steps {
                sh 'pwd'
                sh 'ls -R'
            }
        }

        // 🐳 BUILD IMAGES
        stage('Build Docker Images') {
            steps {
                sh '''
                docker build -t auth-service:${IMAGE_TAG} ./services/auth-service
                docker build -t user-service:${IMAGE_TAG} ./services/user-service
                docker build -t order-service:${IMAGE_TAG} ./services/order-service
                docker build -t notification-service:${IMAGE_TAG} ./services/notification-service
                '''
            }
        }

        // 🔐 LOGIN TO AWS ECR
        stage('Login to ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {
                    sh '''
                    aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_URL
                    '''
                }
            }
        }

        // 📤 PUSH IMAGES
        stage('Push to ECR') {
            steps {
                sh '''
                docker tag auth-service:${IMAGE_TAG} $ECR_URL/auth-service:${IMAGE_TAG}
                docker push $ECR_URL/auth-service:${IMAGE_TAG}

                docker tag user-service:${IMAGE_TAG} $ECR_URL/user-service:${IMAGE_TAG}
                docker push $ECR_URL/user-service:${IMAGE_TAG}

                docker tag order-service:${IMAGE_TAG} $ECR_URL/order-service:${IMAGE_TAG}
                docker push $ECR_URL/order-service:${IMAGE_TAG}

                docker tag notification-service:${IMAGE_TAG} $ECR_URL/notification-service:${IMAGE_TAG}
                docker push $ECR_URL/notification-service:${IMAGE_TAG}
                '''
            }
        }

        // 🚀 DEPLOY USING HELM
        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                helm upgrade my-app ./devops-saas \
                --set auth.image=$ECR_URL/auth-service:${IMAGE_TAG} \
                --set user.image=$ECR_URL/user-service:${IMAGE_TAG} \
                --set order.image=$ECR_URL/order-service:${IMAGE_TAG} \
                --set notification.image=$ECR_URL/notification-service:${IMAGE_TAG}
                '''
            }
        }
    }
}