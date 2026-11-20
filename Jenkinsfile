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
        withCredentials([
            usernamePassword(
                credentialsId: 'dockerhub-login',
                usernameVariable: 'DOCKER_USER',
                passwordVariable: 'DOCKER_TOKEN'
            )
        ]) {
            powershell '''
                $ErrorActionPreference = "Stop"

                Write-Host "Checking Jenkins Docker Hub credentials..."

                $user = $env:DOCKER_USER.Trim()
                $token = $env:DOCKER_TOKEN.Trim()

                if ($user -ne "poorvi2396") {
                    throw "Wrong Docker Hub username in Jenkins credentials"
                }

                if ([string]::IsNullOrWhiteSpace($token)) {
                    throw "Docker Hub token is empty"
                }

                $configDir = Join-Path $env:WORKSPACE ".docker"
                New-Item -ItemType Directory -Force -Path $configDir | Out-Null

                $env:DOCKER_CONFIG = $configDir

                $psi = New-Object System.Diagnostics.ProcessStartInfo
                $psi.FileName = "docker.exe"
                $psi.Arguments = "--config `"$configDir`" login --username `"$user`" --password-stdin"
                $psi.UseShellExecute = $false
                $psi.RedirectStandardInput = $true
                $psi.RedirectStandardOutput = $true
                $psi.RedirectStandardError = $true
                $psi.CreateNoWindow = $true

                $process = New-Object System.Diagnostics.Process
                $process.StartInfo = $psi
                [void]$process.Start()

                $process.StandardInput.WriteLine($token)
                $process.StandardInput.Close()

                $output = $process.StandardOutput.ReadToEnd()
                $errorOutput = $process.StandardError.ReadToEnd()
                $process.WaitForExit()

                if ($process.ExitCode -ne 0) {
                    Write-Host "Docker login returned an error."
                    throw "Docker Hub login failed"
                }

                Write-Host "Docker Hub login successful!"
            '''
        }
    }
}

        // Stage 4: Push Docker Image
        stage('Push Image') {
            steps {
                withEnv(["DOCKER_CONFIG=${env.WORKSPACE}\\.docker"]) {
                    bat '''
                        docker push %DOCKER_IMAGE%:%IMAGE_TAG%
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
