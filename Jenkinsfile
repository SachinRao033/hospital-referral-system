pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-creds',
                    url: 'https://github.com/SachinRao033/hospital-referral-system.git'
            }
        }

        stage('Build Docker Images') {
            steps {
                sh 'docker compose build --no-cache'
            }
        }

        stage('Deploy Containers') {
            steps {
                sh '''
                docker compose down || true
                docker compose up -d
                '''
            }
        }

        stage('Wait for Services') {
            steps {
                sh 'sleep 20'
            }
        }

        stage('Prisma Migration') {
            steps {
                sh '''
                docker exec hospital-backend npx prisma migrate deploy || \
                docker exec hospital-backend npx prisma db push
                '''
            }
        }

        stage('Seed Super Admin') {
            steps {
                sh 'docker exec hospital-backend node src/utils/seedAdmin.js || true'
            }
        }

        stage('Health Check') {
            steps {
                sh 'curl --fail http://localhost:4000/api/health'
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully!'
        }
        failure {
            sh 'docker logs hospital-backend --tail=50 || true'
            echo 'Deployment failed!'
        }
    }
}
