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
                    docker build -t talhahamidsyed/flask-app .
                '''
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    bat 'docker login -u %DOCKER_USER% -p %DOCKER_PASS%'
                    bat 'docker push talhahamidsyed/flask-app'
                }
            }
        }
            
        stage('Deploy to EC2') {
            steps {
                script {
                    sshagent(credentials: ['my-new-key-1']) {
                        def deployCommand = '''
                            docker pull talhahamidsyed/flask-app &&
                            docker stop flask-app || true &&
                            docker rm flask-app || true &&
                            docker run -d --name flask-app -p 80:5000 talhahamidsyed/flask-app
                        '''
        
                        // Use powershell instead of bat to ensure better compatibility
                        powershell """
                            ssh -o StrictHostKeyChecking=no ubuntu@16.171.136.221 \\
                            '${deployCommand.replaceAll("\n", " ")}'
                        """
                    }
                }
            }
        }

    }

    post {
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}

