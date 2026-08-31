pipeline {

    agent any

    stages {

        stage('Checkout') {
            steps {
                echo '======================================'
                echo ' CHECKING OUT SOURCE CODE'
                echo '======================================'

                git branch: 'main',
                    url: 'https://github.com/senavs/login-react-python.git'
            }
        }

        stage('Docker Check') {
            steps {
                echo '======================================'
                echo ' DOCKER CHECK'
                echo '======================================'

                sh 'docker --version'
                sh 'docker compose version'
            }
        }

        stage('Validate Docker Compose') {
            steps {
                echo '======================================'
                echo ' VALIDATING DOCKER COMPOSE'
                echo '======================================'

                sh '''
                    docker compose config
                '''
            }
        }

        stage('Build') {
            steps {
                echo '======================================'
                echo ' BUILDING DOCKER IMAGES'
                echo '======================================'

                sh '''
                    docker compose build
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo '======================================'
                echo ' DEPLOYING APPLICATION'
                echo '======================================'

                sh '''
                    docker compose down --remove-orphans || true

                    docker rm -f login-frontend login-backend login-mysql 2>/dev/null || true

                    docker compose up -d
                '''
            }
        }

        stage('Wait for MySQL') {
            steps {
                echo '======================================'
                echo ' WAITING FOR MYSQL'
                echo '======================================'

                sh '''
                    sleep 15
                '''
            }
        }

        stage('Wait for Applications') {
            steps {
                echo '======================================'
                echo ' WAITING FOR APPLICATIONS'
                echo '======================================'

                sh '''
                    sleep 10
                '''
            }
        }

        stage('Verify') {
            steps {
                echo '======================================'
                echo ' VERIFYING DEPLOYMENT'
                echo '======================================'

                sh '''
                    echo "===== CONTAINERS ====="
                    docker compose ps

                    echo ""
                    echo "===== FRONTEND TEST ====="
                    curl -I --max-time 10 http://localhost:3000 || true

                    echo ""
                    echo "===== BACKEND TEST ====="
                    curl -I --max-time 10 http://localhost:8081 || true

                    echo ""
                    echo "===== MYSQL STATUS ====="
                    docker inspect -f '{{.State.Status}}' login-mysql || true
                '''
            }
        }
    }

    post {

        success {
            echo '''
======================================
 Jenkins Deployment SUCCESS
======================================
'''
        }

        failure {
            echo '''
======================================
 Jenkins Deployment FAILED
======================================
'''
        }

        always {
            sh '''
                echo "===== FINAL STATUS ====="
                docker compose ps || true

                echo ""
                echo "===== FRONTEND LOGS ====="
                docker logs login-frontend --tail 50 2>/dev/null || true

                echo ""
                echo "===== BACKEND LOGS ====="
                docker logs login-backend --tail 50 2>/dev/null || true

                echo ""
                echo "===== MYSQL LOGS ====="
                docker logs login-mysql --tail 50 2>/dev/null || true

                echo ""
                echo "===== DISK SPACE ====="
                df -h /
            '''
        }
    }
}
