pipeline {
    agent any

    environment {
        APP_NAME = 'my-devops-app'
        CONTAINER_NAME = 'my-devops-container'
        RELEASE_DIR = 'release'
        ZIP_NAME = 'release-package.zip'
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

        stage('Build Report') {
            steps {
                bat '''
                echo Build started > build-report.txt
                echo Workspace: %CD% >> build-report.txt
                dir >> build-report.txt
                '''
            }
        }

        stage('Prepare Release Folder') {
            steps {
                bat '''
                if exist %RELEASE_DIR% rmdir /s /q %RELEASE_DIR%
                mkdir %RELEASE_DIR%
                '''
            }
        }

        stage('Test') {
            steps {
                bat '''
                if not exist app.py (
                    echo ERROR: app.py not found
                    exit /b 1
                )
                echo Test passed
                '''
            }
        }

        stage('Copy App Files') {
            steps {
                bat '''
                copy app.py %RELEASE_DIR%\\
                if exist requirements.txt copy requirements.txt %RELEASE_DIR%\\
                if exist Dockerfile copy Dockerfile %RELEASE_DIR%\\
                '''
            }
        }

        stage('Package ZIP') {
            steps {
                bat '''
                powershell -Command "if (Test-Path '%ZIP_NAME%') { Remove-Item '%ZIP_NAME%' -Force }"
                powershell -Command "Compress-Archive -Path '%RELEASE_DIR%\\*' -DestinationPath '%ZIP_NAME%' -Force"
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                bat '''
                if not exist Dockerfile (
                    echo ERROR: Dockerfile not found
                    exit /b 1
                )
                docker build -t %APP_NAME% .
                '''
            }
        }

        stage('Remove Old Container') {
            steps {
                bat '''
                docker rm -f %CONTAINER_NAME% 2>nul
                exit /b 0
                '''
            }
        }

        stage('Run Container') {
            steps {
                bat '''
                docker run -d -p 8088:80 --name %CONTAINER_NAME% %APP_NAME%
                '''
            }
        }

        stage('Deploy Report') {
            steps {
                bat '''
                echo Deployment completed successfully > deploy-report.txt
                docker ps >> deploy-report.txt
                '''
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '*.txt, *.zip', fingerprint: true
        }
    }
}