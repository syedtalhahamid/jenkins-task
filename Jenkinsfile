pipeline {
    agent any

    environment {
        // Use a variable for the image name for easier management
        IMAGE_NAME = 'talhahamidsyed/flask-app'
    }

    stages {
        stage('Build Docker Image') {
            steps {
                // Use sh for Linux agents or bat for Windows agents
                // The %VAR% syntax is for bat, $VAR is for sh.
                bat "docker build -t ${env.IMAGE_NAME} ."
            }
        }

        stage('Push to DockerHub') {
            steps {
                // Your DockerHub login stage is fine
                withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    bat '''
                        echo %PASS% | docker login -u %USER% --password-stdin
                        docker push %IMAGE_NAME%
                    '''
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                // Use the sshagent wrapper with your new credential ID
                sshagent(credentialsId: 'ec2-ssh-key') {
                    // The sshagent wrapper automatically handles the private key.
                    // We add '-o StrictHostKeyChecking=no' to prevent the host key prompt.
                    // We also make the remote docker commands more robust.
                    bat '''
                        ssh -o StrictHostKeyChecking=no ec2-user@16.171.136.221 "docker pull ${env.IMAGE_NAME}:latest && \
                        docker stop flask-app || true && \
                        docker rm flask-app || true && \
                        docker run -d --name flask-app -p 80:5000 ${env.IMAGE_NAME}:latest"
                    '''
                }
            }
        }
    }
}
