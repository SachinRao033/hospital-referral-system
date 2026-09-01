pipeline {
    agent any

    options {
        timestamps()
    }

    environment {
        COMPOSE_PROJECT_NAME = "hospital"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-creds',
                    url: 'https://github.com/SachinRao033/hospital-referral-system.git'
            }
        }

        stage('Create .env Files') {
            steps {
                withCredentials([
                    string(credentialsId: 'backend-env', variable: 'BACKEND_ENV'),
                    string(credentialsId: 'frontend-env', variable: 'FRONTEND_ENV')
                ]) {
                    sh '''
                    printf "%s" "$BACKEND_ENV" > backend/.env
                    printf "%s" "$FRONTEND_ENV" > frontend/.env
                    '''
                }
            }
        }

        stage('Build Images') {
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

        stage('Wait') {
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

        stage('Verify') {
            steps {
                sh 'docker ps'
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Hospital Referral System deployed successfully!'
        }
        failure {
            echo 'FAILED: Deployment failed.'
        }
    }
}
