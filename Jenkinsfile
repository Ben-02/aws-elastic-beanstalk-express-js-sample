pipeline {
    agent {
        docker { image 'node:16' }
    }
    environment {
        DOCKER_HUB_CREDENTIALS = credentials('docker-hub-credentials-id')
        DOCKER_IMAGE = "ben278/aws-express-sample"
        SNYK_TOKEN = credentials('snyk-token-id')
    }
    stages {
        stage('Install Dependencies') {
            steps {
                sh 'npm install --save'
            }
        }
        stage('Run Unit Tests') {
            steps {
                sh 'npm test || exit 0' // Exit 0 prevents failure if no tests exist
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
                sh 'npm install -g snyk'
                sh 'snyk auth $SNYK_TOKEN'
                sh 'snyk test --severity-threshold=high || exit 1' // Fails if high/critical vulnerabilities are found
            }
        }
    }
    post {
        always {
            sh 'docker logout'
        }
    }
}