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
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    bat '''
                        echo %PASS% | docker login -u %USER% --password-stdin
                        docker push %IMAGE_NAME%
                    '''
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(credentials: ['my-new-key-1']) {
                    bat """
                        ssh -o StrictHostKeyChecking=no %EC2_HOST% "docker pull %IMAGE_NAME% && docker stop flask || true && docker rm flask || true && docker run -d -p 5000:5000 --name flask %IMAGE_NAME%"
                    """
                }
            }
        }

    }
}
