pipeline {
    agent any

    environment {
        DB_URL = "jdbc:postgresql://postgres:5432/bankdb"
        DB_USER = "sonar"
        DB_PASS = "sonar123"
        DOCKER_NETWORK = "bindnamed_jenkins_network"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/ferilauw/bank-microservices.git'
            }
        }

        stage('Debug Workspace') {
            steps {
                sh '''
                echo "=== CURRENT DIR ==="
                pwd

                echo "=== ACCOUNT MIGRATION ==="
                ls -la account-service/db/migration || true

                echo "=== TRANSACTION MIGRATION ==="
                ls -la transaction-service/db/migration || true
                '''
            }
        }

        stage('Account Service Scan') {
            steps {
                dir('account-service') {
                    sh 'sonar-scanner'
                }
            }
        }

        stage('Transaction Service Scan') {
            steps {
                dir('transaction-service') {
                    sh 'sonar-scanner'
                }
            }
        }

        stage('DB Migration Account') {
            steps {
                dir('account-service') {
                    sh '''
                    echo "=== RUN FLYWAY ACCOUNT ==="

                    docker run --rm \
                    --network $DOCKER_NETWORK \
                    -v $(pwd)/db/migration:/flyway/sql \
                    flyway/flyway \
                    -url=$DB_URL \
                    -user=$DB_USER \
                    -password=$DB_PASS \
                    migrate
                    '''
                }
            }
        }

        stage('DB Migration Transaction') {
            steps {
                dir('transaction-service') {
                    sh '''
                    echo "=== RUN FLYWAY TRANSACTION ==="

                    docker run --rm \
                    --network $DOCKER_NETWORK \
                    -v $(pwd)/db/migration:/flyway/sql \
                    flyway/flyway \
                    -url=$DB_URL \
                    -user=$DB_USER \
                    -password=$DB_PASS \
                    migrate
                    '''
                }
            }
        }
    }
}