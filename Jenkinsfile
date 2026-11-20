pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'poorvi2396/my-python-app3'
        IMAGE_TAG = 'latest'
        CONTAINER_NAME = 'my-python-container'
        APP_PORT = '5000'
    }

    stages {

        // Stage 1: Build Docker Image
        stage('Build Docker Image') {
            steps {
                bat '''
                    docker build -t %DOCKER_IMAGE%:%IMAGE_TAG% .
                '''
            }
        }

        // Stage 2: Docker Environment Check
        stage('Docker Environment Check') {
            steps {
                powershell '''
                    Write-Host "Windows User:"
                    whoami

                    Write-Host "Docker Version:"
                    docker --version

                    Write-Host "Docker Engine:"
                    docker info --format "{{.ServerVersion}}"
                '''
            }
        }

        // Stage 3: Login to Docker Hub
       

stage('Login to DockerHub') {
    steps {
        withEnv(["DOCKER_CONFIG=${env.WORKSPACE}\\.docker"]) {
            withCredentials([
                usernamePassword(
                    credentialsId: 'dockerhub-pat-new',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_TOKEN'
                )
            ]) {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    New-Item -ItemType Directory -Force `
                        -Path $env:DOCKER_CONFIG | Out-Null

                    $user = $env:DOCKER_USER.Trim()
                    $token = $env:DOCKER_TOKEN.Trim()

                    Write-Host "Docker Hub username: $user"

                    if ($user -ne "poorvi2396") {
                        throw "Incorrect Docker Hub username in Jenkins credentials"
                    }

                    if ([string]::IsNullOrWhiteSpace($token)) {
                        throw "Docker Hub token is empty"
                    }

                    $token | docker --config $env:DOCKER_CONFIG login `
                        --username $user --password-stdin 2>&1

                    if ($LASTEXITCODE -ne 0) {
                        throw "Docker login failed. Check the Docker error above."
                    }

                    Write-Host "Docker Hub login successful!"
                '''
            }
        }
    }
}

        // Stage 4: Push Docker Image

stage('Push Image') {
    steps {
        withEnv(["DOCKER_CONFIG=${env.WORKSPACE}\\.docker"]) {
            bat '''
                docker --config "%DOCKER_CONFIG%" push %DOCKER_IMAGE%:%IMAGE_TAG%

                if errorlevel 1 exit /b 1
            '''
        }
    }
}

        // Stage 5: Deploy Container
        stage('Deploy Container') {
            steps {
                withEnv(["DOCKER_CONFIG=${env.WORKSPACE}\\.docker"]) {
                    bat '''
                        docker stop %CONTAINER_NAME% 2>NUL
                        docker rm %CONTAINER_NAME% 2>NUL

                        docker run -d ^
                            --name %CONTAINER_NAME% ^
                            -p %APP_PORT%:%APP_PORT% ^
                            %DOCKER_IMAGE%:%IMAGE_TAG%

                        docker ps
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully!'
            echo 'Docker image pushed and application deployed.'
        }

        failure {
            echo 'CI/CD Pipeline failed. Check the Console Output.'
        }

        always {
            bat '''
                if exist "%WORKSPACE%\\.docker" (
                    rmdir /s /q "%WORKSPACE%\\.docker"
                )
            '''
            echo 'Pipeline execution completed.'
        }
    }
}
