pipeline {
    agent any

    triggers {
        cron('H/5 * * * *')
    }

    environment {
        APP_NAME       = 'my-devops-app'
        CONTAINER_NAME = 'my-devops-container'
        HOST_PORT      = '8091'
        CONTAINER_PORT = '80'
        RELEASE_DIR    = 'release'
        ZIP_NAME       = 'my-devops-release.zip'
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
                echo Build successful> build-report.txt
                type build-report.txt
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
                if exist index.html (
                    echo Test Passed> test-report.txt
                    type test-report.txt
                ) else (
                    echo index.html file missing
                    exit /b 1
                )

                if exist Dockerfile (
                    echo Dockerfile found
                ) else (
                    echo Dockerfile missing
                    exit /b 1
                )
                '''
            }
        }

        stage('Copy App Files') {
            steps {
                bat '''
                copy /Y index.html %RELEASE_DIR%\\
                copy /Y Dockerfile %RELEASE_DIR%\\
                copy /Y build-report.txt %RELEASE_DIR%\\
                copy /Y test-report.txt %RELEASE_DIR%\\
                dir %RELEASE_DIR%
                '''
            }
        }

        stage('Package ZIP') {
            steps {
                bat '''
                if exist %ZIP_NAME% del /f /q %ZIP_NAME%
                powershell -NoProfile -ExecutionPolicy Bypass -Command "Compress-Archive -Path '.\\%RELEASE_DIR%\\*' -DestinationPath '.\\%ZIP_NAME%' -Force"
                if not exist %ZIP_NAME% (
                    echo ZIP file was not created
                    exit /b 1
                )
                dir
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                bat '''
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
                docker run -d -p %HOST_PORT%:%CONTAINER_PORT% --name %CONTAINER_NAME% %APP_NAME%
                '''
            }
        }

        stage('Deploy Report') {
            steps {
                bat '''
                echo Deployment successful on port %HOST_PORT%> deploy-report.txt
                type deploy-report.txt
                copy /Y deploy-report.txt %RELEASE_DIR%\\
                '''
            }
        }
    }

    post {
        success {
            archiveArtifacts artifacts: '*.txt, *.zip, release/*', fingerprint: true
            echo 'Pipeline completed successfully'
        }
        failure {
            echo 'Pipeline failed'
            bat 'docker ps -a'
        }
        always {
            bat 'dir'
        }
    }
}