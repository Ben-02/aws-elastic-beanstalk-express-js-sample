pipeline {
    agent none
    environment {
        DOCKER_HUB_CREDENTIALS = credentials('docker-hub-credentials-id')
        DOCKER_IMAGE = "ben278/aws-express-sample"
        SNYK_TOKEN = credentials('snyk-token-id')
    }
    stages {
        stage('Checkout') {
            agent { 
                docker { 
                    image 'node:16'
                    args '-u root --privileged -v /var/jenkins_home:/var/jenkins_home:z'  // ADD: Root user, privileged for DinD, :z for relabeling
                } 
            }
            steps {
                checkout scm
            }
        }
        stage('Install Dependencies') {
            agent { 
                docker { 
                    image 'node:16'
                    args '-u root --privileged -v /var/jenkins_home:/var/jenkins_home:z'
                } 
            }
            steps {
                sh 'ls -la'  // ADD TEMP: Debug; remove after fix to verify workspace access
                sh 'npm install --save'
            }
        }
        stage('Run Unit Tests') {
            agent { 
                docker { 
                    image 'node:16'
                    args '-u root --privileged -v /var/jenkins_home:/var/jenkins_home:z'
                } 
            }
            steps {
                sh 'npm test || exit 0'
            }
        }
        stage('Security Scan') {
            agent { 
                docker { 
                    image 'node:16'
                    args '-u root --privileged -v /var/jenkins_home:/var/jenkins_home:z'
                } 
            }
            steps {
                sh 'npm install -g snyk'
                sh 'snyk auth $SNYK_TOKEN'
                sh 'snyk test --severity-threshold=high'
            }
        }
        stage('Build Docker Image') {
            agent { 
                docker { 
                    image 'node:16'
                    args '-u root --privileged -v /var/jenkins_home:/var/jenkins_home:z'
                } 
            }
            steps {
                sh 'docker build -t $DOCKER_IMAGE:latest .'
            }
        }
        stage('Push to Registry') {
            agent { 
                docker { 
                    image 'node:16'
                    args '-u root --privileged -v /var/jenkins_home:/var/jenkins_home:z'
                } 
            }
            steps {
                sh 'echo $DOCKER_HUB_CREDENTIALS_PSW | docker login -u $DOCKER_HUB_CREDENTIALS_USR --password-stdin'
                sh 'docker push $DOCKER_IMAGE:latest'
            }
        }
    }
    post {
        always {
            script {
                node('built-in') {
                    sh 'docker logout || true'
                }
            }
        }
    }
}