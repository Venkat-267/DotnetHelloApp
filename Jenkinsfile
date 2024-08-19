pipeline {
    agent any

    environment {
        // Define environment variables
        EC2_IP = '15.206.185.149'
        APP_NAME = 'HelloWorldApp'
        REPO_URL = 'https://github.com/Venkat-267/DotnetHelloApp.git'
        DEPLOY_DIR = '/home/ubuntu/deployment'
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
                            ssh -o StrictHostKeyChecking=no ubuntu@${EC2_IP} << 'EOF'
                                cd ${DEPLOY_DIR}
                                echo "[Unit]
                                Description=Your .NET Core Application
                                After=network.target

                                [Service]
                                ExecStart=/usr/bin/dotnet ${DEPLOY_DIR}/bin/Release/net8.0/publish/HelloWorldApp.dll --urls http://0.0.0.0:5000
                                WorkingDirectory=${DEPLOY_DIR}/bin/Release/net8.0/publish
                                Restart=always
                                RestartSec=10
                                SyslogIdentifier=your-application
                                User=ubuntu
                                Group=ubuntu
                                Environment=ASPNETCORE_ENVIRONMENT=Production

                                [Install]
                                WantedBy=multi-user.target" | sudo tee /etc/systemd/system/helloapp.service > /dev/null
                                # Reload systemd and start the service
                                sudo systemctl daemon-reload
                                sudo systemctl enable helloapp.service
                                sudo systemctl start helloapp.service
                                sudo systemctl restart nginx
                            EOF
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
