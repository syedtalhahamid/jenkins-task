// pipeline {
//     agent any

//     stages {
//         stage('Git Pull') {
//             steps {
//                 git 'https://github.com/syedtalhahamid/jenkins-task.git'
//             }
//         }

        

//         stage('Docker Build') {
//             steps {
//                 bat 'docker build -t talhahamidsyed/flask .'
//             }
//         }

//         stage('Docker Push') {
//             steps {
//                 withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
//                     bat 'docker login -u %DOCKER_USER% -p %DOCKER_PASS%'
//                     bat 'docker push talhahamidsyed/flask'
//                 }
//             }
//         }

//         stage('Deploy Flask via SSM') {
//             steps {
//                 withCredentials([
//                     [$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-cred-id']
//                 ]) {
//                     powershell '''
//                         # Step 1: Define commands array as a string for inline JSON
//                         $cmds = '["docker pull talhahamidsyed/flask","docker rm -f flask || true","docker run -d --name flask -p 80:5000 talhahamidsyed/flask"]'
        
//                         # Step 2: Use AWS CLI with correct --parameters key=value format
//                         aws ssm send-command `
//                             --document-name "AWS-RunShellScript" `
//                             --comment "DeployFlask" `
//                             --instance-ids "i-0eb4223f049a2edf2" `
//                             --parameters "commands=$cmds" `
//                             --region "eu-north-1"
//                     '''
//                 }
//             }
//         }


//     }
// }








pipeline {
    agent any

    environment {
        IMAGE_NAME = 'talhahamidsyed/flask'
    }

    stages {
        stage('Git Pull') {
            steps {
                git 'https://github.com/syedtalhahamid/jenkins-task.git'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $IMAGE_NAME
                    '''
                }
            }
        }

        stage('Deploy Flask via SSM') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-cred-id']
                ]) {
                    sh '''
                        aws ssm send-command \
                            --document-name "AWS-RunShellScript" \
                            --comment "Deploy Flask" \
                            --instance-ids "i-0eb4223f049a2edf2" \
                            --parameters 'commands=["docker pull talhahamidsyed/flask", "docker rm -f flask || true", "docker run -d --name flask -p 80:5000 talhahamidsyed/flask"]' \
                            --region eu-north-1
                    '''
                }
            }
        }
    }
}
