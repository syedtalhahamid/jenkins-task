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
                    # 1. Define the commands you want to run on the EC2 instance.
                    $commandsToRun = @(
                        "docker pull talhahamidsyed/flask",
                        "docker rm -f flask || true",
                        "docker run -d --name flask -p 80:5000 talhahamidsyed/flask"
                    )

                    # 2. Create a PowerShell object that matches the JSON structure required by the --parameters argument.
                    $parametersObject = @{
                        commands = $commandsToRun
                    }

                    # 3. Convert the PowerShell object into a compact, valid JSON string.
                    # This is the safest way to generate the JSON.
                    $jsonString = $parametersObject | ConvertTo-Json -Compress -Depth 4

                    # 4. Manually build and execute the command.
                    # The crucial part is wrapping the $jsonString variable in single quotes ('$jsonString').
                    # This tells PowerShell to pass the entire, complex JSON string as a single argument to aws.exe,
                    # protecting all the quotes and special characters inside it.
                    aws ssm send-command `
                      --document-name "AWS-RunShellScript" `
                      --comment "Deploying flask via Jenkins" `
                      --instance-ids "i-0eb4223f049a2edf2" `
                      --parameters '$jsonString' `
                      --region "eu-north-1"
                '''
            }
        }
    }    
}
