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
                    "docker rm -f flask || true",
                    "docker run -d --name flask -p 80:5000 talhahamidsyed/flask"
                )

                # Build JSON string manually for AWS CLI
                $json = @{
                    commands = $commands
                } | ConvertTo-Json -Compress

                # Use aws cli directly with correct parameter format
                aws ssm send-command `
                    --document-name "AWS-RunShellScript" `
                    --comment "Deploy Flask" `
                    --instance-ids "i-0eb4223f049a2edf2" `
                    --parameters @{commands=$commands} `
                    --region "eu-north-1"
            '''
        }
    }
}

    }
}
