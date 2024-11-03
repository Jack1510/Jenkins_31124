pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        S3_BUCKET = 'jenkins031124'
        APP_NAME = 'Jenkins0311'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', credentialsId: 'github-credentials', url: 'https://github.com/Jack1510/Jenkins_31124.git'
            }
        }

        stage('Package') {
            steps {
                script {
                    sh 'zip -r app.zip ./*'         // Package all files in the directory
                }
            }
        }

        stage('Upload to S3') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-access', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh """
                    aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                    aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                    aws s3 cp app.zip s3://$S3_BUCKET/app.zip --region $AWS_REGION
                    """
                }
            }
        }

        stage('Deploy to AWS Elastic Beanstalk') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-access', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh """
                    aws elasticbeanstalk create-application-version --application-name $APP_NAME \
                    --version-label ${env.BUILD_ID} --source-bundle S3Bucket=$S3_BUCKET,S3Key=app.zip --region $AWS_REGION

                    aws elasticbeanstalk update-environment --application-name $APP_NAME \
                    --environment-name your-env-name --version-label ${env.BUILD_ID} --region $AWS_REGION
                    """
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up workspace...'
            cleanWs()
        }
        success {
            echo 'Deployment succeeded!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}
