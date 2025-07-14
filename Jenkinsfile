pipeline {
    agent any // You can define a specific agent label if you have multiple agents, for example: label 'your-windows-jenkins-agent'

    stages {
        stage('Git Pull') {
            steps {
                git 'https://github.com/syedtalhahamid/jenkins-task.git'
            }
        }
        stage('Test PowerShell') {
            steps {
                // Keep this as 'powershell' since you're on a Windows agent
                powershell 'Write-Output "PowerShell is working!"'
            }
        }    
        
        stage('Docker Build') {
            steps {
                // Ensure Docker is installed and configured on the Windows Jenkins agent
                bat '''
                    docker build -t talhahamidsyed/flask .
                '''
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
        powershell '''
            # Define the commands to run on the EC2 instance
            $commands = @(
                "docker pull talhahamidsyed/flask",
                "docker rm -f flask; exit 0",
                "docker run -d --name flask -p 80:5000 talhahamidsyed/flask"
            )

            # Convert to JSON and escape double quotes
            $params = @{ commands = $commands }
            $json = $params | ConvertTo-Json -Compress
            $escapedJson = "`"{0}`"" -f $json.Replace('"', '\\"')

            # Build argument list
            $args = @(
                "ssm", "send-command",
                "--document-name", "AWS-RunShellScript",
                "--comment", "`"Deploying flask via Jenkins`"",
                "--instance-ids", "i-0eb4223f049a2edf2",
                "--parameters", $escapedJson,
                "--region", "eu-north-1"
            )

            # Run the AWS CLI command
            Start-Process -FilePath "aws" -ArgumentList $args -NoNewWindow -Wait
        '''
    }
}

    }    
}
