pipeline {
    agent any

    environment {
        COMPOSE_PROJECT_NAME = 'developer-portfolio'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Repository') {
            steps {
                sh '''
                    echo "=== Repository Structure ==="
                    ls -la
                    echo "=== App_Server ==="
                    ls -la App_Server
                    echo "=== Portfolio ==="
                    ls -la Portfolio
                '''
            }
        }

        stage('Prepare Environment') {
            steps {
                withCredentials([
                    string(credentialsId: 'portfolio-mysql-root-password', variable: 'MYSQL_ROOT_PASSWORD'),
                    string(credentialsId: 'portfolio-mysql-user', variable: 'MYSQL_USER'),
                    string(credentialsId: 'portfolio-mysql-password', variable: 'MYSQL_PASSWORD'),
                    string(credentialsId: 'portfolio-session-secret', variable: 'SESSION_SECRET'),
                    string(credentialsId: 'portfolio-admin-password', variable: 'ADMIN_PASSWORD'),
                    string(credentialsId: 'portfolio-admin-email', variable: 'ADMIN_EMAIL')
                ]) {
                    sh '''
                        set +x

                        cat > .env <<EOF
MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
MYSQL_USER=${MYSQL_USER}
MYSQL_PASSWORD=${MYSQL_PASSWORD}
SESSION_SECRET=${SESSION_SECRET}
ADMIN_PASSWORD=${ADMIN_PASSWORD}
ADMIN_EMAIL=${ADMIN_EMAIL}
EOF

                        chmod 600 .env

                        echo "Production environment file created."
                    '''
                }
            }
        }

        stage('Validate Compose') {
            steps {
                sh '''
                    echo "=== Validating Docker Compose ==="
                    docker compose config -q
                    echo "Docker Compose configuration is valid."
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    echo "=== Building Docker Images ==="
                    docker compose build
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    echo "=== Stopping Existing Deployment ==="
                    docker compose down || true

                    echo "=== Starting Deployment ==="
                    docker compose up -d
                '''
            }
        }

        stage('Verify Containers') {
            steps {
                sh '''
                    echo "=== Container Status ==="
                    docker compose ps

                    echo "=== Waiting for Application ==="
                    sleep 20

                    echo "=== Backend Logs ==="
                    docker compose logs --tail=50 backend

                    echo "=== Frontend Logs ==="
                    docker compose logs --tail=30 frontend
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    echo "=== Checking Frontend Container ==="

                    FRONTEND_IP=$(docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' portfolio-frontend)

                    echo "Frontend container IP: ${FRONTEND_IP}"

                    docker run --rm \
                        --network developer-portfolio_default \
                        curlimages/curl:latest \
                        -f http://portfolio-frontend/

                    echo ""
                    echo "=== Checking Backend Container ==="

                    docker run --rm \
                        --network developer-portfolio_default \
                        curlimages/curl:latest \
                        -f http://portfolio-backend:8000/

                    echo ""
                    echo "=== Deployment Verification Complete ==="
                '''
            }
        }
    }

    post {

        success {
            echo 'Developer Portfolio deployment successful!'
        }

        failure {
            echo 'Developer Portfolio deployment failed.'

            sh '''
                docker compose ps || true
                docker compose logs --tail=100 || true
            '''
        }

        always {
            sh '''
                rm -f .env || true
                docker compose ps || true
            '''
        }
    }
}
