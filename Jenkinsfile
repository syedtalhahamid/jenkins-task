pipeline {
    agent any

    stages {
        stage('Git Pull') {
            steps {
                git 'https://github.com/syedtalhahamid/jenkins-task.git'
            }
        }

        stage('Test PowerShell') {
            steps {
                powershell 'Write-Output "PowerShell is working!"'
            }
        }

        stage('Docker Build') {
            steps {
                bat 'docker build -t talhahamidsyed/flask .'
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    bat 'docker login -u %DOCKER_USER% -p %DOCKER_PASS%'
                    bat 'docker push talhahamidsyed/flask'
                }
            }
        }

     stage('Deploy Flask via SSM') {
    steps {
        withCredentials([
            [$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-cred-id']
        ]) {
            powershell '''
                $commands = @(
                    "docker pull talhahamidsyed/flask",
                    "docker rm -f flask ; exit 0",
                    "docker run -d --name flask -p 80:5000 talhahamidsyed/flask"
                )

                $params = @{ "commands" = $commands }
                $jsonParams = $params | ConvertTo-Json -Compress

                # Write params to a file to avoid escape hell
                Set-Content -Path "params.json" -Value $jsonParams

                # Build args as array, avoiding parser issues
                $args = @(
                    "ssm", "send-command",
                    "--document-name", "AWS-RunShellScript",
                    "--comment", "DeployFlask",
                    "--instance-ids", "i-0eb4223f049a2edf2",
                    "--parameters", (Get-Content params.json -Raw),
                    "--region", "eu-north-1"
                )

                # Start process
                Start-Process "aws" -ArgumentList $args -NoNewWindow -Wait
            '''
        }
    }
}

    
    }
}
