pipeline {
    agent any

    triggers {
        cron('H/5 * * * *')
    }

    environment {
        APP_NAME       = 'my-devops-app'
        CONTAINER_NAME = 'my-devops-container'
        HOST_PORT      = '8090'
        CONTAINER_PORT = '80'
        RELEASE_DIR    = 'release'
        ZIP_NAME       = 'my-devops-release.zip'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                echo 'Checking out source code from GitHub...'
                checkout scm
            }
        }

        stage('Check Files') {
            steps {
                bat 'dir'
            }
        }

        stage('Prepare Release Folder') {
            steps {
                bat """
                if exist %RELEASE_DIR% rmdir /s /q %RELEASE_DIR%
                mkdir %RELEASE_DIR%
                """
            }
        }

        stage('Build Report') {
            steps {
                bat """
                echo Build successful > build-report.txt
                type build-report.txt
                copy build-report.txt %RELEASE_DIR%\\build-report.txt
                """
            }
        }

        stage('Test') {
            steps {
                bat """
                if exist index.html (
                    echo Test Passed > test-report.txt
                    type test-report.txt
                    copy test-report.txt %RELEASE_DIR%\\test-report.txt
                ) else (
                    echo Test Failed
                    exit /b 1
                )
                """
            }
        }

        stage('Copy App Files') {
            steps {
                bat """
                copy index.html %RELEASE_DIR%\\index.html
                copy Dockerfile %RELEASE_DIR%\\Dockerfile
                """
            }
        }

        stage('Package ZIP') {
            steps {
                bat """
                if exist %ZIP_NAME% del /f /q %ZIP_NAME%
                powershell -Command "Compress-Archive -Path %RELEASE_DIR%\\* -DestinationPath %ZIP_NAME% -Force"
                """
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %APP_NAME% .'
            }
        }

        stage('Remove Old Container') {
            steps {
                bat 'docker rm -f %CONTAINER_NAME% || exit /b 0'
            }
        }

        stage('Run Container') {
            steps {
                bat 'docker run -d -p %HOST_PORT%:%CONTAINER_PORT% --name %CONTAINER_NAME% %APP_NAME%'
            }
        }

        stage('Deploy Report') {
            steps {
                bat """
                echo Deployment successful > deploy-report.txt
                type deploy-report.txt
                copy deploy-report.txt %RELEASE_DIR%\\deploy-report.txt
                """
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
        }
        always {
            bat 'docker ps -a'
            echo 'Pipeline finished'
        }
    }
}