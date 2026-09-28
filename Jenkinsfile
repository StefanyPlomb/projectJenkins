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
        IMAGE_NAME = 'stefanyplombon/projectjenkins'
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

        stage('Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub',
                                                  usernameVariable: 'DH_USER',
                                                  passwordVariable: 'DH_PASS')]) {
                    sh '''
                        echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${IMAGE_NAME}:latest
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                // :latest local é a mesma imagem enviada ao Docker Hub; recria o web se mudou
                sh 'docker compose -p projectjenkins -f docker-compose.yml up -d web'
            }
        }
    }

    post {
        always {
            // remove só a tag numerada local; :latest continua em uso pelo web
            sh 'docker image rm ${IMAGE_NAME}:${IMAGE_TAG} || true'
        }
    }
}
