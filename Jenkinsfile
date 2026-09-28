pipeline {
    agent any

    options {
        disableConcurrentBuilds()
    }

    triggers {
        // Verifica o repositório a cada ~2 minutos e roda se houver commit novo
        pollSCM('H/2 * * * *')
    }

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
                sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} -t ${IMAGE_NAME}:latest .'
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

        stage('Deploy') {
            steps {
                // recria o container web se a imagem :latest mudou
                sh 'docker compose -p projectjenkins -f docker-compose.yml up -d web'
            }
        }
    }

    post {
        always {
            // remove só a tag numerada; :latest continua em uso pelo web
            sh 'docker image rm ${IMAGE_NAME}:${IMAGE_TAG} || true'
        }
    }
}
