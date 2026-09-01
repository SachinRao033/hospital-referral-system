pipeline {
    agent any

    options {
        timestamps()
    }

    stages {

        stage('Update Project') {
            steps {
                sh '''
                cd /home/ubuntu/hospital-referral-system
                git pull origin main
                '''
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                cd /home/ubuntu/hospital-referral-system
                docker compose build --no-cache
                '''
            }
        }

        stage('Deploy Containers') {
            steps {
                sh '''
                cd /home/ubuntu/hospital-referral-system
                docker compose down
                docker compose up -d
                '''
            }
        }

        stage('Prisma Migration') {
            steps {
                sh '''
                docker exec hospital-backend npx prisma migrate deploy
                '''
            }
        }

        stage('Seed Admin') {
            steps {
                sh '''
                docker exec hospital-backend npm run seed
                '''
            }
        }

        stage('Verify') {
            steps {
                sh 'docker ps'
            }
        }
    }

    post {
        success {
            echo 'Hospital Referral System deployed successfully!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}
