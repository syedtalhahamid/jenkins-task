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
                    # This is the most robust way to handle JSON and external commands in PowerShell.

                    # 1. Define the commands as a PowerShell array.
                    $commands = @(
                        "docker pull talhahamidsyed/flask",
                        "docker rm -f flask || true",
                        "docker run -d --name flask -p 80:5000 talhahamidsyed/flask"
                    )

                    # 2. Create the parameter object.
                    $parametersObject = @{
                        commands = $commands
                    }

                    # 3. Convert the object to a valid JSON string.
                    # The -Depth parameter is a good practice to ensure all levels are converted.
                    $jsonParameters = $parametersObject | ConvertTo-Json -Compress -Depth 4

                    # 4. Use "Splatting" to pass arguments safely to the external aws.exe command.
                    # We create a hashtable where keys are the parameter names (without the --)
                    # and values are the parameter values. This avoids all command-line quoting issues.
                    $awsSsmParams = @{
                        DocumentName = "AWS-RunShellScript"
                        Comment      = "Deploying flask via Jenkins"
                        InstanceIds  = "i-0eb4223f049a2edf2"
                        Parameters   = $jsonParameters # Pass the complete, correct JSON string here
                        Region       = "eu-north-1"
                    }

                    # 5. Execute the command. The @awsSsmParams tells PowerShell to use the hashtable for the arguments.
                    aws ssm send-command @awsSsmParams
                '''
            }
        }
    }    
}
