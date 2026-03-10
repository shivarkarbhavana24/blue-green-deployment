pipeline {
    agent any
    environment {
        BLUE_TG = "arn:aws:elasticloadbalancing:us-east-1:123456789012:targetgroup/Blue-TG/abc123"
        GREEN_TG = "arn:aws:elasticloadbalancing:us-east-1:123456789012:targetgroup/Green-TG/def456"
        ALB_LISTENER = "arn:aws:elasticloadbalancing:us-east-1:123456789012:listener/app/BlueGreen-ALB/xyz987"
    }
    stages {
        stage('Deploy to Inactive') {
            steps {
                script {
                    // Determine inactive environment
                    def activeTG = sh(script: "aws elbv2 describe-listeners --listener-arn $ALB_LISTENER --query 'Listeners[0].DefaultActions[0].TargetGroupArn' --output text", returnStdout: true).trim()
                    def inactiveTG = activeTG == BLUE_TG ? GREEN_TG : BLUE_TG

                    echo "Deploying to inactive environment: ${inactiveTG}"
                    sh """
                        scp -i ~/.ssh/key.pem -o StrictHostKeyChecking=no index.html ubuntu@<inactive-instance-ip>:/var/www/html/
                    """
                }
            }
        }
        stage('Health Check') {
            steps {
                script {
                    def status = sh(script: "curl -f http://<inactive-instance-ip>", returnStatus: true)
                    if (status != 0) {
                        error "Health check failed!"
                    }
                }
            }
        }
        stage('Switch Traffic') {
            steps {
                script {
                    sh """
                        aws elbv2 modify-listener --listener-arn $ALB_LISTENER --default-actions Type=forward,TargetGroupArn=$inactiveTG
                    """
                    echo "Traffic switched to inactive environment"
                }
            }
        }
    }
    post {
        failure {
            script {
                // Rollback traffic to previous environment
                sh """
                    aws elbv2 modify-listener --listener-arn $ALB_LISTENER --default-actions Type=forward,TargetGroupArn=$activeTG
                """
                echo "Rollback executed: Traffic reverted to previous environment"
            }
        }
    }
}