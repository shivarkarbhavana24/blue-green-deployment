pipeline {
    agent any
    environment {
        BLUE_TG = "arn:aws:elasticloadbalancing:us-east-1:246816819870:targetgroup/Blue-TG/5267a746109b3117"
        GREEN_TG = "arn:aws:elasticloadbalancing:us-east-1:246816819870:targetgroup/Green-TG/7610e67f973b0ec4"
        ALB_LISTENER = "arn:aws:elasticloadbalancing:us-east-1:246816819870:listener/app/BlueGreen-ALB/a7fbdf4879e40f40/981e0ff257f3a38b"
        ACTIVE_TG = "${BLUE_TG}"  // initial active TG
    }
    stages {
        stage('Deploy to Inactive') {
            steps {
                script {
                    // Determine inactive TG
                    def inactiveTG = (ACTIVE_TG == BLUE_TG) ? GREEN_TG : BLUE_TG
                    echo "Deploying to inactive environment: ${inactiveTG}"
                    
                    // Deploy code via SCP to inactive instance
                    sh "scp -i ~/.ssh/bluekey.pem -o StrictHostKeyChecking=no index.html ubuntu@<inactive-instance-ip>:/var/www/html/"

                    // Save inactive TG for traffic switch
                    env.INACTIVE_TG = inactiveTG
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
                        aws elbv2 modify-listener --listener-arn $ALB_LISTENER \
                        --default-actions Type=forward,TargetGroupArn=$INACTIVE_TG
                    """
                    echo "Traffic switched to inactive environment"
                    
                    // Update ACTIVE_TG after switch
                    env.ACTIVE_TG = env.INACTIVE_TG
                }
            }
        }
    }

    post {
        failure {
            script {
                echo "Rollback: Traffic revert to previous active TG ($ACTIVE_TG)"
                sh """
                    aws elbv2 modify-listener --listener-arn $ALB_LISTENER \
                    --default-actions Type=forward,TargetGroupArn=$ACTIVE_TG
                """
            }
        }
    }
}