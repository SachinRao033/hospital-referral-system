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

        stage('Create Environment Files') {
            steps {
                sh '''
                cat > backend/.env << EOF
DATABASE_URL=mysql://root:root@mysql:3306/hospital_referral
JWT_SECRET=SachinHospital123
PORT=4000
FRONTEND_URL=http://13.205.210.92:3000
ALLOWED_ORIGINS=http://13.205.210.92:3000
EOF

                cat > frontend/.env << EOF
VITE_API_URL=http://13.205.210.92:4000/api
EOF
                '''
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
                docker compose down --remove-orphans || true
                docker compose up -d
                '''
            }
        }

        stage('Wait for Services') {
            steps {
                sh '''
                echo "Waiting for services to start..."
                sleep 20
                docker ps
                '''
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
                sh '''
                docker exec hospital-backend node src/utils/seedAdmin.js || true
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                curl --fail http://localhost:4000/api/health
                '''
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
            sh 'docker logs hospital-backend --tail=50 || true'
            sh 'docker logs hospital-frontend --tail=30 || true'
            echo 'Deployment failed!'
        }
    }
}
