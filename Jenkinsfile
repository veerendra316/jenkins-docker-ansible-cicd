pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REGISTRY = '592011499817.dkr.ecr.us-east-1.amazonaws.com'
        ECR_REPOSITORY = 'jenkins-cicd-web'
        IMAGE_TAG = "${BUILD_NUMBER}"
        ECR_IMAGE = "${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t jenkins-cicd-web:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Docker Test') {
            steps {
                sh '''
                    docker rm -f jenkins-cicd-test || true

                    docker run -d \
                      --name jenkins-cicd-test \
                      -p 8081:80 \
                      jenkins-cicd-web:${BUILD_NUMBER}

                    sleep 5

                    curl -f http://localhost:8081
                '''
            }
        }

        stage('Push to ECR') {
            steps {
                sh '''
                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login --username AWS --password-stdin ${ECR_REGISTRY}

                    docker tag jenkins-cicd-web:${BUILD_NUMBER} ${ECR_IMAGE}

                    docker push ${ECR_IMAGE}

                    docker tag jenkins-cicd-web:${BUILD_NUMBER} \
                      ${ECR_REGISTRY}/${ECR_REPOSITORY}:latest

                    docker push \
                      ${ECR_REGISTRY}/${ECR_REPOSITORY}:latest
                '''
            }
        }

        stage('Deploy with Ansible') {
            steps {
                sh '''
                    cd ansible
                    ansible-playbook -i inventory deploy.yml
                '''
            }
        }
    }

    post {
        always {
            sh 'docker rm -f jenkins-cicd-test || true'
        }
    }
}
