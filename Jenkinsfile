pipeline {
    agent any

    environment {
        SERVER_IMAGE = 'imensaied/mern-server'
        CLIENT_IMAGE = 'imensaied/mern-client'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verifier Docker') {
            steps {
                bat 'docker --version'
                bat 'docker compose version'
            }
        }

        stage('Build images') {
            steps {
                bat 'docker build -t %SERVER_IMAGE%:latest ./Server'
                bat 'docker build -t %CLIENT_IMAGE%:latest ./Client'
            }
        }

                stage('Push vers Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-imensaied',
                                                  usernameVariable: 'DH_USER',
                                                  passwordVariable: 'DH_PASS')]) {
                    bat 'docker login -u %DH_USER% -p %DH_PASS% docker.io'
                    bat 'docker push %SERVER_IMAGE%:latest'
                    bat 'docker push %CLIENT_IMAGE%:latest'
                }
            }
        }

        stage('Deploy avec Docker Compose') {
            steps {
                bat 'docker compose down'
                bat 'docker compose up -d --build'
                bat 'docker ps'
            }
        }
    }

    post {
        success {
            echo 'Pipeline termine avec succes'
        }
        failure {
            echo 'Pipeline echoue, verifier les logs'
        }
    }
}