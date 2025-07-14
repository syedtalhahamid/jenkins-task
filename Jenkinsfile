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
                        # Set environment variables for AWS CLI
                        $env:AWS_ACCESS_KEY_ID = "$env:AWS_ACCESS_KEY_ID"
                        $env:AWS_SECRET_ACCESS_KEY = "$env:AWS_SECRET_ACCESS_KEY"

                        # Define commands to execute on EC2
                        $commands = @(
                            "docker pull talhahamidsyed/flask",
                            "docker rm -f flask; exit 0",
                            "docker run -d --name flask -p 80:5000 talhahamidsyed/flask"
                        )

                        # Create parameters object and convert to JSON
                        $params = @{ commands = $commands }
                        $json = $params | ConvertTo-Json -Compress -Depth 3
                        $escapedJson = $json.Replace('"', '\\"')

                        # Build the full command as a string
                        $command = @"
aws ssm send-command `
  --document-name "AWS-RunShellScript" `
  --comment "Deploying flask via Jenkins" `
  --instance-ids "i-0eb4223f049a2edf2" `
  --parameters "$escapedJson" `
  --region "eu-north-1"
"@

                        # Execute the command
                        iex $command
                    '''
                }
            }
        }
    }
}
