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

        stage('Test') {
            steps {
                sh 'docker image inspect jenkins-cicd-web:${BUILD_NUMBER}'
            }
        }
    }
}
