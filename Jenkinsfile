pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
        S3_BUCKET          = 'my-static-site-bucket'  // Replace with your bucket name
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate HTML') {
            steps {
                echo 'Validating HTML...'
                // Basic check — ensure index.html exists
                sh 'test -f index.html && echo "index.html found" || exit 1'
            }
        }

        stage('Deploy to S3') {
            steps {
                withAWS(credentials: 'aws-credentials') {
                    sh """
                        aws s3 sync . s3://${S3_BUCKET}/ \
                            --exclude ".git/*" \
                            --exclude "Jenkinsfile" \
                            --exclude "*.md" \
                            --delete
                    """
                }
            }
        }

        stage('Invalidate CloudFront (optional)') {
            when { expression { return fileExists('cloudfront-config.json') } }
            steps {
                withAWS(credentials: 'aws-credentials') {
                    script {
                        def cfId = readFile('cloudfront-config.json').trim()
                        sh "aws cloudfront create-invalidation --distribution-id ${cfId} --paths '/*'"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Deployment to S3://${S3_BUCKET} completed successfully."
        }
        failure {
            echo "Deployment failed."
        }
    }
}
