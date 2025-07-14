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
                    # 1. Define the commands to run on the EC2 instance.
                    $commandsToRun = @(
                        "docker pull talhahamidsyed/flask",
                        "docker rm -f flask || true",
                        "docker run -d --name flask -p 80:5000 talhahamidsyed/flask"
                    )

                    # 2. Create the PowerShell object for the parameters.
                    $parametersObject = @{
                        commands = $commandsToRun
                    }

                    # 3. Convert the object into a compact, valid JSON string.
                    $jsonString = $parametersObject | ConvertTo-Json -Compress -Depth 4

                    # 4. THE CRITICAL STEP: Create a new string where every double-quote (")
                    # is replaced with an escaped double-quote (\"). This makes the string safe
                    # to be passed on the command line.
                    $escapedJsonString = $jsonString.Replace('"', '\"')

                    # 5. Build and execute the command.
                    # We use double quotes around the variable ("$escapedJsonString") so that PowerShell
                    # expands it to its content. Because the content now has escaped quotes, the
                    # command will be parsed correctly by the AWS CLI.
                    aws ssm send-command `
                      --document-name "AWS-RunShellScript" `
                      --comment "Deploying flask via Jenkins" `
                      --instance-ids "i-0eb4223f049a2edf2" `
                      --parameters "$escapedJsonString" `
                      --region "eu-north-1"
                '''
            }
        }
    }    
}
