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

        stage('Verify Deployment') {
            steps {
                sh '''
                    echo "=== Container Status ==="
                    docker compose ps

                    echo "=== Waiting for Application ==="
                    sleep 15

                    echo "=== Testing Frontend ==="
                    curl -f http://localhost:8084/

                    echo ""
                    echo "=== Testing Backend Through Nginx ==="
                    curl -f http://localhost:8084/api/ || true

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
                docker compose ps || true
            '''
        }
    }
}
