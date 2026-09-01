pipeline {
    agent any

    options {
        timestamps()
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-creds',
                    url: 'https://github.com/SachinRao033/hospital-referral-system.git'
            }
        }

        stage('Create Environment Files') {
            steps {
                withCredentials([
                    string(credentialsId: 'BACKEND_ENV', variable: 'BACKEND_ENV'),
                    string(credentialsId: 'FRONTEND_ENV', variable: 'FRONTEND_ENV')
                ]) {
                    sh '''
                    printf "%s" "$BACKEND_ENV" > backend/.env
                    printf "%s" "$FRONTEND_ENV" > frontend/.env
                    '''
                }
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
                sh 'docker exec hospital-backend npx prisma migrate deploy'
            }
        }

        stage('Seed Admin') {
            steps {
                sh 'docker exec hospital-backend npm run seed'
            }
        }

        stage('Verify Deployment') {
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
