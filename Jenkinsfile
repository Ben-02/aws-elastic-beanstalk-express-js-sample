pipeline {
    agent {
        docker {
            image 'docker:dind'
            args '-v /var/run/docker.sock:/var/run/docker.sock --privileged -v /snap/bin/docker:/usr/bin/docker'
        }
    }
    environment {
        DOCKER_HUB_CREDENTIALS = credentials('docker-hub-credentials-id')
        DOCKER_IMAGE = "Ben278/aws-express-sample"
        SNYK_TOKEN = credentials('snyk-token-id')
        DOCKER_HOST = 'tcp://dind:2376'
        DOCKER_TLS_VERIFY = '1'
        DOCKER_CERT_PATH = '/certs/client'
    }
    stages {
        stage('Install Dependencies') {
            steps {
                sh 'docker run --rm -v $(pwd):/app node:16 npm install --save'
            }
        }
        stage('Run Unit Tests') {
            steps {
                sh 'docker run --rm -v $(pwd):/app node:16 npm test'
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
                sh 'docker run --rm -v $(pwd):/app node:16 sh -c "npm install -g snyk && snyk auth --token=$SNYK_TOKEN && snyk test --severity-threshold=high"'
            }
        }
    }
    post {
        always {
            sh 'docker logout || true'
        }
    }
}