pipeline {
    agent any

    environment {
        IMAGE_NAME = 'projectjenkins'
        IMAGE_TAG  = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
            }
        }

        stage('Check') {
            steps {
                sh 'docker run --rm ${IMAGE_NAME}:${IMAGE_TAG} python manage.py check'
            }
        }

        stage('Test') {
            steps {
                sh 'docker run --rm ${IMAGE_NAME}:${IMAGE_TAG} python manage.py test'
            }
        }
    }

    post {
        always {
            sh 'docker image rm ${IMAGE_NAME}:${IMAGE_TAG} || true'
        }
    }
}
