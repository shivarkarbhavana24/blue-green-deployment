pipeline {
    agent any

    stages {

        stage('Pull Code') {
            steps {
                git branch: 'main', url: 'https://github.com/shivarkarbhavana24/blue-green-deployment.git'
            }
        }

        stage('Deploy to Green') {
            steps {
                sh '''
                scp -o StrictHostKeyChecking=no index.html ubuntu@54.198.90.69:/var/www/html/
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                curl -f http://54.198.90.69
                '''
            }
        }

        stage('Switch Traffic to Green') {
            steps {
                sh '''
                aws elbv2 modify-listener \
                --listener-arn arn:aws:elasticloadbalancing:us-east-1:246816819870:listener/app/BlueGreen-ALB/a7fbdf4879e40f40/981e0ff257f3a38b \
                --default-actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:us-east-1:246816819870:targetgroup/Green-TG/7610e67f973b0ec4
                '''
            }
        }
    }

    post {
        failure {
            steps {
                sh '''
                aws elbv2 modify-listener \
                --listener-arn arn:aws:elasticloadbalancing:us-east-1:246816819870:listener/app/BlueGreen-ALB/a7fbdf4879e40f40/981e0ff257f3a38b \
                --default-actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:us-east-1:246816819870:targetgroup/Blue-TG/5267a746109b3117
                '''
            }
        }
    }
}