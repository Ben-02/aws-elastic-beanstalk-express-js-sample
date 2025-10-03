pipeline {
    agent any // Run on Jenkins master with DinD
    environment {
        DOCKER_HUB_CREDENTIALS = credentials('docker-hub-credentials-id')
        DOCKER_IMAGE = "Ben278/aws-express-sample"
        SNYK_TOKEN = credentials('snyk-token-id')
        DOCKER_HOST = 'tcp://dind:2376' // Point to DinD
        DOCKER_TLS_VERIFY = '1'
        DOCKER_CERT_PATH = '/certs/client'
    }
    stages {
        stage('Install Dependencies') {
            steps {
                script {
                    docker.image('node:16').inside {
                        sh 'npm install --save'
                    }
                }
            }
        }
        stage('Run Unit Tests') {
            steps {
                script {
                    docker.image('node:16').inside {
                        sh 'npm test || exit 0'
                    }
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:latest .'
            }
        }
        stage('Push to Registry') {
            steps {
                sh 'echo $DOCKER_HUB_CREDENTIALS_PSW | docker login -u $DOCKER_HUB_CREDENTIALS_USR --password-stdin'
                sh 'docker push $DOCKER_IMAGE:latest'
            }
        }
        stage('Security Scan') {
            steps {
                script {
                    docker.image('node:16').inside {
                        sh 'npm install -g snyk'
                        sh 'snyk auth $SNYK_TOKEN'
                        sh 'snyk test --severity-threshold=high || exit 1'
                    }
                }
            }
        }
    }
    post {
        always {
            sh 'docker logout'
        }
    }
}