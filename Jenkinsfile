pipeline {
    agent any

    environment {
        EC2_HOST = 'ec2-user@16.171.136.221' 
        IMAGE_NAME = 'talhahamidsyed/flask-app'
    }

    stages {

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %IMAGE_NAME% .'
            }
        }

        stage('Push to DockerHub') {
            steps {
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
        // Use full path to ssh.exe (assuming OpenSSH is installed in default path)
        bat '"C:\\Windows\\System32\\OpenSSH\\ssh.exe" -i "C:\\Users\\Team Codenera\\Downloads\\my-new-key-1.pem" ec2-user@16.171.136.221 "docker pull talhahamidsyed/flask-app && docker run -d -p 80:5000 talhahamidsyed/flask-app"'
    }
}


    }
}
