pipeline {
    agent any

    environment {
        // Define environment variables
        EC2_IP = '15.207.112.177'
        APP_NAME = 'HelloWorldApp'
        REPO_URL = 'https://github.com/Venkat-267/DotnetHelloApp.git'
        DEPLOY_DIR = '/home/ubuntu/DotnetHelloApp'
        APP_PORT = '5000'
        CREDENTIALS_ID = 'ec2-ssh-credentials'
    }

    stages {

        stage('Build') {
            steps {
                script {
                    // Build and publish the .NET Core application
                    sh 'dotnet publish -c Release'
                }
            }
        }

        stage('Deploy') {
            steps {
                sshagent(credentials: [CREDENTIALS_ID]) {
                    script {
                        // Copy the published application to the EC2 instance
                        sh '''
                            scp -o StrictHostKeyChecking=no -r bin/Release/net8.0/publish ubuntu@${EC2_IP}:${DEPLOY_DIR}
                        '''

                        // SSH into the EC2 instance and run deployment commands
                        sh '''
                            ssh -o StrictHostKeyChecking=no ubuntu@${EC2_IP} "
                                cd ${DEPLOY_DIR}

                                sudo systemctl daemon-reload
                                sudo systemctl restart helloapp
                                sudo systemctl restart nginx
                            "
                        '''
                    }
                }
            }
        }

        stage('Post-Deployment') {
            steps {
                // Archive deployment logs or other relevant files
                archiveArtifacts artifacts: 'deploy-logs/**/*', allowEmptyArchive: true
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed.'
        }
    }
}
