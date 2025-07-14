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
                // Ensure PowerShell is available on your Jenkins agent
                powershell 'Write-Output "PowerShell is working!"' 
            }
        }    
        
        stage('Docker Build') {
            steps {
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
                    // Use the SSH agent step to manage the SSH key securely
                    // Replace 'your-ec2-ssh-credential-id' with the actual credential ID from Jenkins
                    sshagent(credentials: ['my-new-key-1']) {
                        // The following command will be executed on the Jenkins agent,
                        // which then connects to the EC2 instance via SSH
                        sh "ssh -o StrictHostKeyChecking=no ubuntu@16.171.136.221 \"docker pull talhahamidsyed/flask-app && docker rm -f flask-app || true && docker run -d --name flask-app -p 80:5000 talhahamidsyed/flask-app\""
                    }
                }
            }
        }

    }
}
