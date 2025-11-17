pipeline {
    agent any

    environment {
        DOCKER_CREDS = credentials('docker-hub')
    }

    stages {
        stage('Clone Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/Meghana2417/PaymentService.git'
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build --no-cache -t meghana1724/paymentservice:latest ."
            }
        }

        stage('Docker Login') {
            steps {
                sh 'echo "$DOCKER_CREDS_PSW" | docker login -u "$DOCKER_CREDS_USR" --password-stdin'
            }
        }

        stage('Docker Push') {
            steps {
                sh "docker push meghana1724/paymentservice:latest"
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                docker stop paymentservice || true
                docker rm paymentservice || true

                docker pull meghana1724/paymentservice:latest

                docker run -d --name paymentservice -p 8006:8006 meghana1724/paymentservice:latest
                '''
            }
        }
    }
}
