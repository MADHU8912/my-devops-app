pipeline {
    agent any

    triggers {
        cron('H/5 * * * *')
    }

    environment {
        APP_NAME       = 'my-devops-app'
        CONTAINER_NAME = 'my-devops-container'
        RELEASE_DIR    = 'release'
        ZIP_NAME       = 'my-devops-release.zip'
        PORT           = '8090'
    }

    stages {

        stage('Checkout SCM') {
            steps {
                echo 'Checking out source code from GitHub...'
                checkout scm
            }
        }

        stage('Checkout Info') {
            steps {
                bat '''
                echo ===== GIT INFO ===== > scm-report.txt
                git --version >> scm-report.txt 2>&1
                git branch >> scm-report.txt 2>&1
                git remote -v >> scm-report.txt 2>&1
                git log -1 --oneline >> scm-report.txt 2>&1
                type scm-report.txt
                '''
            }
        }

        stage('Check Files') {
            steps {
                bat '''
                echo ===== WORKSPACE FILES ===== > file-report.txt
                dir >> file-report.txt
                type file-report.txt
                '''
            }
        }

        stage('Build Report') {
            steps {
                bat '''
                echo ===== BUILD REPORT ===== > build-report.txt
                echo Build Number: %BUILD_NUMBER% >> build-report.txt
                echo Job Name: %JOB_NAME% >> build-report.txt
                echo Workspace: %WORKSPACE% >> build-report.txt
                echo Date: %DATE% %TIME% >> build-report.txt
                echo. >> build-report.txt
                echo Files in workspace: >> build-report.txt
                dir >> build-report.txt
                type build-report.txt
                '''
            }
        }

        stage('Prepare Release Folder') {
            steps {
                bat '''
                if exist %RELEASE_DIR% (
                    rmdir /s /q %RELEASE_DIR%
                )
                mkdir %RELEASE_DIR%
                echo Release folder prepared successfully > prepare-release.txt
                type prepare-release.txt
                '''
            }
        }

        stage('Test') {
            steps {
                bat '''
                echo ===== TEST STAGE ===== > test-report.txt

                if exist Dockerfile (
                    echo Dockerfile found >> test-report.txt
                ) else (
                    echo ERROR: Dockerfile not found >> test-report.txt
                    type test-report.txt
                    exit /b 1
                )

                if exist app.py (
                    echo app.py found >> test-report.txt
                ) else (
                    echo app.py not found, checking other app files... >> test-report.txt
                )

                if exist app.txt (
                    echo app.txt found >> test-report.txt
                )

                if exist index.html (
                    echo index.html found >> test-report.txt
                )

                if exist requirements.txt (
                    echo requirements.txt found >> test-report.txt
                )

                echo Test stage completed >> test-report.txt
                type test-report.txt
                '''
            }
        }

        stage('Copy App Files') {
            steps {
                bat '''
                echo ===== COPY FILES ===== > copy-report.txt

                if exist Dockerfile copy Dockerfile %RELEASE_DIR%\\ >> copy-report.txt
                if exist Jenkinsfile copy Jenkinsfile %RELEASE_DIR%\\ >> copy-report.txt
                if exist app.py copy app.py %RELEASE_DIR%\\ >> copy-report.txt
                if exist app.txt copy app.txt %RELEASE_DIR%\\ >> copy-report.txt
                if exist index.html copy index.html %RELEASE_DIR%\\ >> copy-report.txt
                if exist requirements.txt copy requirements.txt %RELEASE_DIR%\\ >> copy-report.txt

                if exist src xcopy src %RELEASE_DIR%\\src\\ /E /I /Y >> copy-report.txt 2>&1
                if exist public xcopy public %RELEASE_DIR%\\public\\ /E /I /Y >> copy-report.txt 2>&1
                if exist dist xcopy dist %RELEASE_DIR%\\dist\\ /E /I /Y >> copy-report.txt 2>&1

                echo Files copied successfully >> copy-report.txt
                dir %RELEASE_DIR% >> copy-report.txt
                type copy-report.txt
                '''
            }
        }

        stage('Package ZIP') {
            steps {
                bat '''
                echo ===== PACKAGE ZIP ===== > zip-report.txt

                powershell -Command "if (Test-Path '%ZIP_NAME%') { Remove-Item '%ZIP_NAME%' -Force }"
                powershell -Command "Compress-Archive -Path '%RELEASE_DIR%\\*' -DestinationPath '%ZIP_NAME%' -Force"

                if exist %ZIP_NAME% (
                    echo ZIP package created successfully >> zip-report.txt
                ) else (
                    echo ERROR: ZIP package was not created >> zip-report.txt
                    type zip-report.txt
                    exit /b 1
                )

                type zip-report.txt
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                bat '''
                echo ===== DOCKER BUILD ===== > docker-build-report.txt
                docker --version >> docker-build-report.txt 2>&1

                docker build -t %APP_NAME% . >> docker-build-report.txt 2>&1

                if %ERRORLEVEL% NEQ 0 (
                    echo ERROR: Docker image build failed >> docker-build-report.txt
                    type docker-build-report.txt
                    exit /b 1
                )

                echo Docker image built successfully >> docker-build-report.txt
                type docker-build-report.txt
                '''
            }
        }

        stage('Remove Old Container') {
            steps {
                bat '''
                echo ===== REMOVE OLD CONTAINER ===== > remove-container-report.txt
                docker rm -f %CONTAINER_NAME% >> remove-container-report.txt 2>&1
                echo Old container remove step completed >> remove-container-report.txt
                type remove-container-report.txt
                exit /b 0
                '''
            }
        }

        stage('Run Container') {
            steps {
                bat '''
                echo ===== RUN CONTAINER ===== > run-container-report.txt

                docker run -d -p %PORT%:80 --name %CONTAINER_NAME% %APP_NAME% >> run-container-report.txt 2>&1

                if %ERRORLEVEL% NEQ 0 (
                    echo ERROR: Container run failed >> run-container-report.txt
                    type run-container-report.txt
                    exit /b 1
                )

                docker ps >> run-container-report.txt 2>&1
                echo Container started successfully >> run-container-report.txt
                type run-container-report.txt
                '''
            }
        }

        stage('Deploy Report') {
            steps {
                bat '''
                echo ===== DEPLOY REPORT ===== > deploy-report.txt
                echo App Name: %APP_NAME% >> deploy-report.txt
                echo Container Name: %CONTAINER_NAME% >> deploy-report.txt
                echo Port: %PORT% >> deploy-report.txt
                echo URL: http://localhost:%PORT% >> deploy-report.txt
                docker ps >> deploy-report.txt 2>&1
                type deploy-report.txt
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
            echo 'Pipeline failed. Check stage logs and generated report files.'
        }
    }
}