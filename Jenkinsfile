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
            string(credentialsId: 'AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'),
            string(credentialsId: 'AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY')
        ]) {
            powershell '''
                # Set AWS credentials
                $env:AWS_ACCESS_KEY_ID = "$env:AWS_ACCESS_KEY_ID"
                $env:AWS_SECRET_ACCESS_KEY = "$env:AWS_SECRET_ACCESS_KEY"

                # Define the docker commands to run on EC2
                $commands = @(
                    "docker pull talhahamidsyed/flask",
                    "docker rm -f flask; exit 0",
                    "docker run -d --name flask -p 80:5000 talhahamidsyed/flask"
                )

                # Create JSON parameters object
                $params = @{ commands = $commands }
                $json = $params | ConvertTo-Json -Compress

                # Call AWS CLI with properly quoted arguments
                & aws ssm send-command `
                    --document-name "AWS-RunShellScript" `
                    --comment "Deploying flask" `
                    --instance-ids "i-0eb4223f049a2edf2" `
                    --parameters $json `
                    --region "eu-north-1"
            '''
        }
    }
}

        
    }
}
