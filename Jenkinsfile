pipeline {
    agent any

    environment {
        // Define environment variables
        EC2_IP = 'http://15.206.185.149/'
        APP_NAME = 'helloapp'
        REPO_URL = 'https://github.com/Venkat-267/DotnetHelloApp.git'
        DEPLOY_DIR = '/home/ubuntu/deployment'
        APP_PORT = '5000'
        CREDENTIALS_ID = 'ec2-ssh-credentials'
    }

    stages {
        stage('Checkout') {
            steps {
                // Clone the GitHub repository
                git url: "${REPO_URL}"
            }
        }

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
                        sh """
                            scp -o StrictHostKeyChecking=no -i $SSH_KEY -r bin/Release/net8.0/publish ubuntu@${EC2_IP}:${DEPLOY_DIR}
                        """

                        // SSH into the EC2 instance and run deployment commands
                        sh """
                            ssh -o StrictHostKeyChecking=no -i $SSH_KEY ubuntu@${EC2_IP} \\
                            'cd ${DEPLOY_DIR} && dotnet HelloWorldApp.dll --urls http://0.0.0.0:5000'
                        """
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
