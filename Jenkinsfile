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
    ```groovy
stage('Deploy: Roll Instances') {
    steps {
        sh '''
        set -e

        echo "Checking existing instance refresh..."

        STATUS=$(aws autoscaling describe-instance-refreshes \
          --auto-scaling-group-name tomcat-app-asg \
          --region ap-south-1 \
          --query 'InstanceRefreshes[0].Status' \
          --output text)

        echo "Current status: $STATUS"

        if [ "$STATUS" = "InProgress" ]; then
            echo "An instance refresh is already running."
            echo "Waiting for it to complete..."

            while true
            do
                STATUS=$(aws autoscaling describe-instance-refreshes \
                  --auto-scaling-group-name tomcat-app-asg \
                  --region ap-south-1 \
                  --query 'InstanceRefreshes[0].Status' \
                  --output text)

                echo "Refresh status: $STATUS"

                case "$STATUS" in
                    Successful)
                        echo "Existing refresh completed successfully."
                        break
                        ;;
                    Failed|Cancelled|RollbackFailed)
                        echo "Existing refresh failed: $STATUS"
                        exit 1
                        ;;
                    *)
                        sleep 30
                        ;;
                esac
            done

        else
            echo "Starting new instance refresh..."

            REFRESH_ID=$(aws autoscaling start-instance-refresh \
              --auto-scaling-group-name tomcat-app-asg \
              --region ap-south-1 \
              --preferences '{"MinHealthyPercentage":50,"InstanceWarmup":120}' \
              --query 'InstanceRefreshId' \
              --output text)

            echo "Started refresh: $REFRESH_ID"

            while true
            do
                STATUS=$(aws autoscaling describe-instance-refreshes \
                  --auto-scaling-group-name tomcat-app-asg \
                  --region ap-south-1 \
                  --instance-refresh-ids "$REFRESH_ID" \
                  --query 'InstanceRefreshes[0].Status' \
                  --output text)

                echo "Refresh status: $STATUS"

                case "$STATUS" in
                    Successful)
                        echo "Deployment completed successfully."
                        break
                        ;;
                    Failed|Cancelled|RollbackFailed)
                        echo "Deployment failed: $STATUS"
                        exit 1
                        ;;
                    *)
                        sleep 30
                        ;;
                esac
            done
        fi
        '''
    }
}
```

  }
}
