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
    agent any // You can define a specific agent label if you have multiple agents, for example: label 'your-ec2-jenkins-agent'

    stages {
        stage('Git Pull') {
            steps {
                git 'https://github.com/syedtalhahamid/jenkins-task.git'
            }
        }
        stage('Test PowerShell') {
            steps {
                // If your Jenkins agent is a Windows EC2 instance, use 'powershell'.
                // If it's a Linux EC2 instance, you would use 'sh' and a different command.
                // Assuming a Windows EC2 instance for this example based on the previous interaction.
                powershell 'Write-Output "PowerShell is working!"'
            }
        }    
        
        stage('Docker Build') {
            steps {
                // This command will be executed on the Jenkins agent (likely an EC2 instance itself)
                // Ensure Docker is installed and configured on this Jenkins agent
                bat '''
                    docker build -t talhahamidsyed/flask-app .
                '''
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    // This command will also be executed on the Jenkins agent
                    bat 'docker login -u %DOCKER_USER% -p %DOCKER_PASS%'
                    bat 'docker push talhahamidsyed/flask-app'
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                script {
                    // Using the 'sshagent' step to manage the SSH key securely.
                    // Replace 'my-new-key-1' with the actual credential ID from Jenkins.
                    // This credential should be of type 'SSH Username with Private Key' {Link: according to Sunflower Lab https://www.thesunflowerlab.com/jenkins-aws-ec2-instance-ssh/}.
                    sshagent(credentials: ['my-new-key-1']) {
                        // The following command will be executed on the Jenkins agent,
                        // which then connects to the EC2 instance via SSH
                        sh "ssh -o StrictHostKeyChecking=no ubuntu@16.171.136.221 \"docker pull talhahamidsyed/flask-app && docker rm -f flask-app || true && docker run -d --name flask-app -p 80:5000 talhahamidsyed/flask-app\"" //
                    }
                }
            }
        }
    }
}
