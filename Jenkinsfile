pipeline {
    agent any

    environment {
        AWS_CREDENTIALS = credentials('aws-credentials-id') // The ID of the AWS credentials you added
        AWS_REGION = 'ap-south-1' // Your AWS region (e.g., us-west-2)
        EB_APP_NAME = 'Sample' // Your Elastic Beanstalk application name
        EB_ENV_NAME = 'Sample-env' // Your Elastic Beanstalk environment name
    }

    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/Venkat-267/DotnetHelloApp.git' // Replace with your Git repository URL
            }
        }

        stage('Publish') {
            steps {
                script {
                    bat 'dotnet publish -c Release -o ./publish' // Publish the application
                }
            }
        }

        stage('Package Application') {
            steps {
                script {
                    zip zipFile: 'HelloWorldApp.zip', archive: false, dir: 'publish' // Create a ZIP package
                }
            }
        }

        stage('Deploy to Elastic Beanstalk') {
            steps {
                withAWS(credentials: "${AWS_CREDENTIALS_ID}", region: "${AWS_REGION}") {
                    s3Upload(file: 'HelloWorldApp.zip', bucket: 'jenkins-venkat-eb', path: 'HelloWorldApp/HelloWorldApp.zip') // Upload to S3

                    // Deploy to Elastic Beanstalk
                    bat """
                    aws elasticbeanstalk create-application-version --application-name ${EB_APP_NAME} --version-label v${BUILD_NUMBER} --source-bundle S3Bucket="jenkins-venkat-eb",S3Key="HelloWorldApp/HelloWorldApp.zip"
                    aws elasticbeanstalk update-environment --environment-name ${EB_ENV_NAME} --version-label v${BUILD_NUMBER}
                    """
                }
            }
        }
    }

    post {
        always {
            cleanWs() // Clean the workspace after the build
        }
    }
}
