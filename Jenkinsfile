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
                // Use powershell for better and safer JSON handling on Windows agents
                powershell '''
                    # Create an array of the commands you want to run
                    $commands = @(
                        "docker pull talhahamidsyed/flask",
                        "docker rm -f flask || true",
                        "docker run -d --name flask -p 80:5000 talhahamidsyed/flask"
                    )

                    # Create a PowerShell object (hashtable) for the parameters
                    $parameters = @{
                        commands = $commands
                    }

                    # Convert the PowerShell object to a compact JSON string.
                    # This is the safest way to build the JSON.
                    $jsonParameters = $parameters | ConvertTo-Json -Compress

                    # Call the aws cli, passing the properly formatted JSON string
                    aws ssm send-command `
                      --document-name "AWS-RunShellScript" `
                      --comment "Deploying flask via Jenkins" `
                      --instance-ids i-0eb4223f049a2edf2 `
                      --parameters $jsonParameters `
                      --region eu-north-1
                '''
            }
        }

    }    
}
