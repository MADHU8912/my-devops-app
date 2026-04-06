pipeline {
    agent any

    environment {
        IMAGE_NAME = "my-devops-app"
        CONTAINER_NAME = "my-devops-container"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Check Files') {
            steps {
                bat 'dir'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %IMAGE_NAME% .'
            }
        }

        stage('Remove Old Container') {
            steps {
                bat 'docker rm -f %CONTAINER_NAME% 2>nul'
            }
        }

        stage('Run Container') {
            steps {
                bat 'docker run -d -p 8090:80 --name %CONTAINER_NAME% %IMAGE_NAME%'
            }
        }
    }

    post {
        always {
            bat 'echo Build finished > build-log.txt'
            archiveArtifacts artifacts: '*.txt', fingerprint: true
        }
    }
}