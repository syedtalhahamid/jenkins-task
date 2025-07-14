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
                        # Define commands for EC2
                        $commands = @(
                            "docker pull talhahamidsyed/flask",
                            "docker rm -f flask || true",
                            "docker run -d --name flask -p 80:5000 talhahamidsyed/flask"
                        )
        
                        # Create a proper JSON structure
                        $jsonObject = @{
                            commands = $commands
                        }
        
                        # Convert to valid JSON (AWS expects keys in double quotes)
                        $jsonString = $jsonObject | ConvertTo-Json -Compress -Depth 2
        
                        # Surround the whole JSON string with single quotes so it works in command line
                        $finalParams = "'$jsonString'"
        
                        # Call AWS SSM using AWS CLI and pass parameters properly
                        aws ssm send-command `
                            --document-name "AWS-RunShellScript" `
                            --comment "Flask deployment" `
                            --instance-ids "i-0eb4223f049a2edf2" `
                            --parameters $finalParams `
                            --region "eu-north-1"
                    '''
                }
            }
        }
    
    }
}
