pipeline {
    agent any

    stages {

        stage('Pull Code') {
            steps {
                git 'https://github.com/shivarkarbhavana24/blue-green-deployment.git'
            }
        }

        stage('Deploy to Green') {
            steps {
                sh '''
                scp -o StrictHostKeyChecking=no index.html ubuntu@GREEN-IP:/var/www/html/
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh 'curl -f http://GREEN-IP'
            }
        }

        stage('Switch Traffic') {
            steps {
                sh '''
                aws elbv2 modify-listener \
                --listener-arn LISTENER-ARN \
                --default-actions Type=forward,TargetGroupArn=GREEN-TG-ARN
                '''
            }
        }
    }

    post {
        failure {
            sh '''
            aws elbv2 modify-listener \
            --listener-arn LISTENER-ARN \
            --default-actions Type=forward,TargetGroupArn=BLUE-TG-ARN
            '''
        }
    }
}