// pipeline {
//     agent any

//     stages {
//         stage('Git Pull') {
//             steps {
//                 git 'https://github.com/syedtalhahamid/jenkins-task.git'
//             }
//         }
//         stage('Test PowerShell') {
//             steps {
//                 powershell 'Write-Output "PowerShell is working!"'
//             }
//         }    
        
//         stage('Docker Build') {
//             steps {
//                 bat '''
//                     docker build -t talhahamidsyed/flask-app .
//                 '''
//             }
//         }

//         stage('Docker Push') {
//             steps {
//                 withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
//                     bat 'docker login -u %DOCKER_USER% -p %DOCKER_PASS%'
//                     bat 'docker push talhahamidsyed/flask-app'
//                 }
//             }
//         }

//         stage('Deploy to EC2') {
//             steps {
//                 bat '''
//                 "C:\\Windows\\System32\\OpenSSH\\ssh.exe" -i "C:\\Users\\Team Codenera\\.ssh\\my-new-key-1.pem" ubuntu@16.171.136.221 ^
//                 "docker pull talhahamidsyed/flask-app && docker rm -f flask-app || true && docker run -d --name flask-app -p 80:5000 talhahamidsyed/flask-app"
//                 '''
//             }
//         }

//     }
// }


// pipeline {
//     agent any

//     stages {
//         stage('Git Pull') {
//             steps {
//                 git 'https://github.com/syedtalhahamid/jenkins-task.git'
//             }
//         }
//         stage('Test PowerShell') {
//             steps {
//                 // Ensure PowerShell is available on your Jenkins agent
//                 powershell 'Write-Output "PowerShell is working!"' 
//             }
//         }    
        
//         stage('Docker Build') {
//             steps {
//                 bat '''
//                     docker build -t talhahamidsyed/flask-app .
//                 '''
//             }
//         }

//         stage('Docker Push') {
//             steps {
//                 withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
//                     bat 'docker login -u %DOCKER_USER% -p %DOCKER_PASS%'
//                     bat 'docker push talhahamidsyed/flask-app'
//                 }
//             }
//         }

//         stage('Deploy to EC2') {
//             steps {
//                 script {
//                     // Use the SSH agent step to manage the SSH key securely
//                     // Replace 'your-ec2-ssh-credential-id' with the actual credential ID from Jenkins
//                     sshagent(credentials: ['my-new-key-1']) {
//                         // The following command will be executed on the Jenkins agent,
//                         // which then connects to the EC2 instance via SSH
//                         sh "ssh -o StrictHostKeyChecking=no ubuntu@16.171.136.221 \"docker pull talhahamidsyed/flask-app && docker rm -f flask-app || true && docker run -d --name flask-app -p 80:5000 talhahamidsyed/flask-app\""
//                     }
//                 }
//             }
//         }

//     }
// }

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
            
        stage('Deploy to EC2 via SSM') {
            steps {
                script {
                    // Define the Docker commands to be executed on the EC2 instance
                    def dockerCommands = """
                        #!/bin/bash
                        docker pull talhahamidsyed/flask
                        docker rm -f flask || true
                        docker run -d --name flask -p 80:5000 talhahamidsyed/flask
                    """

                    // Use AWS CLI to send the command to the EC2 instance via SSM Run Command
                    // Ensure AWS CLI is configured on the Jenkins agent with permissions to use SSM
                    bat """
                        aws ssm send-command ^
                            --instance-ids i-0eb4223f049a2edf2 ^ 
                            --document-name "AWS-RunShellScript" ^ 
                            --comment "Deploying flask via Jenkins" ^ 
                            --parameters commands="${dockerCommands}" ^ 
                            --region eu-north-1
                    """
                }
            }
        }
    }
}
