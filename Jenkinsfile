pipeline {
    agent any
parameters {
        choice(
            name: 'ENV',
            choices: ['dev', 'qa', 'prod'],
            description: 'Select environment to deploy'
        )
    }
environment {
        IMAGE_NAME = "devops-demo"
        IMAGE_TAG  = "${BUILD_NUMBER}"
        EC2_USER   = "ec2-user"
        EC2_IP     = "<EC2_PUBLIC_IP>"
    }
stages {
stage('Checkout Code') {
            steps {
                git branch: 'main', url: '<GITHUB_REPO_URL>'
            }
        }
stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
                """
            }
        }
stage('Manual Approval for PROD') {
            when {
                expression { params.ENV == 'prod' }
            }
            steps {
                input message: 'Approve deployment to PROD?', ok: 'Deploy'
            }
        }
stage('Deploy to EC2') {
            steps {
                sh """
                ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_IP} '
                    docker rm -f ${IMAGE_NAME} || true
                    docker run -d -p 80:80 --name ${IMAGE_NAME} ${IMAGE_NAME}:latest
                '
               """
            }
        }
}
post {
        success {
            echo "Deployment to ${params.ENV} completed successfully"
        }
        failure {
            echo "Deployment failed"
        }
    }
}
