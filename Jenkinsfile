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
                    [$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']
                ]) {
                    powershell '''
                        # Step 1: Define shell commands for EC2 to run
                        $commands = @(
                            "docker pull talhahamidsyed/flask",
                            "docker rm -f flask || true",
                            "docker run -d --name flask -p 80:5000 talhahamidsyed/flask"
                        )
        
                        # Step 2: Manually construct valid JSON string for AWS CLI
                        $json = '{\\"commands\\":[\\"' + ($commands -join '\\",\\"') + '\\"]}'
        
                        # Step 3: Send the SSM command
                        aws ssm send-command `
                            --document-name "AWS-RunShellScript" `
                            --comment "DeployFlask" `
                            --instance-ids "i-0eb4223f049a2edf2" `
                            --parameters $json `
                            --region "eu-north-1"
                    '''
                }
            }
        }


    }
}
