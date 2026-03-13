pipeline {
agent any

environment {
    AWS_REGION = "ap-south-11"
    ECR_REPO = "778265708016.dkr.ecr.ap-south-1.amazonaws.com/my-app"
    ECR_REGISTRY = "778265708016.dkr.ecr.ap-south-1.amazonaws.com"
    IMAGE_TAG = "${BUILD_NUMBER}"
}

stages {

    stage('Clone Repository') {
        steps {
            git branch: 'main',
            url: 'https://github.com/UmamaheswariG-Cloud/java-web-application.git'
        }
    }

    stage('Maven Build') {
        steps {
            sh 'mvn clean package'
        }
    }

    stage('SonarQube Code Scan') {
        steps {
            withSonarQubeEnv('sonar-qube') {
                sh '''
                mvn sonar:sonar \
                -Dsonar.projectKey=java-web-application \
                -Dsonar.sources=backend,frontend,src
                '''
            }
        }
    }

    stage('Login to ECR') {
        steps {
            sh '''
            aws ecr get-login-password --region $AWS_REGION \
            | docker login --username AWS --password-stdin $ECR_REGISTRY
            '''
        }
    }

    stage('Build Docker Images') {
        steps {
            sh '''
            docker build -t $ECR_REPO:backend-$IMAGE_TAG backend/
            docker build -t $ECR_REPO:frontend-$IMAGE_TAG frontend/
            '''
        }
    }

    stage('Push Images to ECR') {
        steps {
            sh '''
            docker push $ECR_REPO:backend-$IMAGE_TAG
            docker push $ECR_REPO:frontend-$IMAGE_TAG
            '''
        }
    }

    stage('Trivy Security Scan') {
        steps {
            sh '''
            echo "Scanning backend image..."
            trivy image --severity HIGH,CRITICAL $ECR_REPO:backend-$IMAGE_TAG || true

            echo "Scanning frontend image..."
            trivy image --severity HIGH,CRITICAL $ECR_REPO:frontend-$IMAGE_TAG || true
            '''
        }
    }

    stage('Deploy to Kubernetes') {
        steps {
            sh '''
            kubectl apply -f jenkins.yaml
            '''
        }
    }
}

}
