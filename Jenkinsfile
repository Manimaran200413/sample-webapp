pipeline {
  agent any
  environment {
    AWS_REGION   = 'ap-south-1'
    ECR_REGISTRY = '210806258948.dkr.ecr.ap-south-1.amazonaws.com'
    ECR_REPO     = '210806258948.dkr.ecr.ap-south-1.amazonaws.com/tomcat-app'
    ASG_NAME     = 'tomcat-app-asg'
  }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Build WAR') {
      steps { sh 'mvn clean package' }
    }
    stage('Build & Push Docker Image') {
      steps {
        // Changed to double quotes ( """) so Jenkins expands the variables
        sh """
          aws ecr get-login-password --region ${AWS_REGION} | \
            docker login --username AWS --password-stdin ${ECR_REGISTRY}
          docker build -t ${ECR_REPO}:${BUILD_NUMBER} -t ${ECR_REPO}:latest .
          docker push ${ECR_REPO}:${BUILD_NUMBER}
          docker push ${ECR_REPO}:latest
        """
      }
    }
stage('Deploy: Roll Instances') {
    steps {
        sh '''
        aws autoscaling describe-instance-refreshes \
  --auto-scaling-group-name tomcat-app-asg \
  --region ap-south-1 \
  --query 'InstanceRefreshes[0].[InstanceRefreshId,Status,PercentageComplete]' \
  --output table
        '''
    }
}
  }
}
