pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t jenkins-cicd-web:${BUILD_NUMBER} .'
            }
        }

        stage('Docker Test') {
            steps {
                sh '''
                    docker rm -f jenkins-cicd-test || true
                    docker run -d --name jenkins-cicd-test -p 8081:80 jenkins-cicd-web:${BUILD_NUMBER}
                    sleep 5
                    curl -f http://localhost:8081
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
