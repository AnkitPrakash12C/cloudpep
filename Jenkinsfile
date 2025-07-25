pipeline {
  agent any
  environment {
    AWS_DEFAULT_REGION = 'us-east-1'
    ECR_REPO = '<your-aws-account-id>.dkr.ecr.us-east-1.amazonaws.com/flask-app'
    IMAGE_TAG = 'latest'
  }
  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }
    stage('Build Docker Image') {
      steps {
        script {
          docker.build("${ECR_REPO}:${IMAGE_TAG}")
        }
      }
    }
    stage('Login to ECR') {
      steps {
        script {
          sh 'aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $ECR_REPO'
        }
      }
    }
    stage('Push Docker Image') {
      steps {
        script {
          sh 'docker push $ECR_REPO:$IMAGE_TAG'
        }
      }
    }
    stage('Terraform Apply') {
      steps {
        dir('terraform') {
          withCredentials([string(credentialsId: 'AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'),
                           string(credentialsId: 'AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY')]) {
            sh 'terraform init'
            sh 'terraform apply -auto-approve -var="app_image=$ECR_REPO:$IMAGE_TAG"'
          }
        }
      }
    }
  }
}