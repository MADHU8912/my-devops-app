pipeline {
    agent any

    environment {
        IMAGE_NAME = "my-devops-app"
        CONTAINER_NAME = "my-devops-container"
        RELEASE_DIR = "release"
        ZIP_FILE = "my-devops-release.zip"
        PORT = "8090"
    }

    options {
        timestamps()
    }

    triggers {
        cron('H/5 * * * *')
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Check Files') {
            steps {
                bat '''
                echo ===== WORKSPACE FILES =====
                dir
                '''
            }
        }

        stage('Build Report') {
            steps {
                bat '''
                echo Build started > build-report.txt
                echo Job Name: %JOB_NAME% >> build-report.txt
                echo Build Number: %BUILD_NUMBER% >> build-report.txt
                echo Workspace: %CD% >> build-report.txt
                echo ========================== >> build-report.txt
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
                if not exist app.txt (
                    echo ERROR: app.txt not found
                    exit /b 1
                )

                if not exist Dockerfile (
                    echo ERROR: Dockerfile not found
                    exit /b 1
                )

                if not exist Jenkinsfile (
                    echo ERROR: Jenkinsfile not found
                    exit /b 1
                )

                echo All required files found > test-report.txt
                type app.txt >> test-report.txt
                '''
            }
        }

        stage('Copy App Files') {
            steps {
                bat '''
                copy app.txt %RELEASE_DIR%\\app.txt
                copy Dockerfile %RELEASE_DIR%\\Dockerfile
                copy Jenkinsfile %RELEASE_DIR%\\Jenkinsfile
                '''
            }
        }

        stage('Package ZIP') {
            steps {
                bat '''
                powershell -Command "if (Test-Path '%ZIP_FILE%') { Remove-Item '%ZIP_FILE%' -Force }"
                powershell -Command "Compress-Archive -Path '%RELEASE_DIR%\\*' -DestinationPath '%ZIP_FILE%' -Force"
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                bat '''
                docker --version
                docker build -t %IMAGE_NAME% .
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
                docker run -d -p %PORT%:80 --name %CONTAINER_NAME% %IMAGE_NAME%
                '''
            }
        }

        stage('Deploy Report') {
            steps {
                bat '''
                echo Deployment successful > deploy-report.txt
                echo Container Name: %CONTAINER_NAME% >> deploy-report.txt
                echo Image Name: %IMAGE_NAME% >> deploy-report.txt
                echo Port: %PORT% >> deploy-report.txt
                echo ========================== >> deploy-report.txt
                docker ps -a >> deploy-report.txt
                '''
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '*.txt, *.zip', fingerprint: true
        }
        success {
            echo 'Pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed. Check stage logs.'
        }
    }
}