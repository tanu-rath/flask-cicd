pipeline {
    agent any

    environment {
        REPO_URL = 'https://github.com/YOUR_USERNAME/flask-ci-cd.git'
        DOCKER_IMAGE = 'YOUR_DOCKERHUB_USERNAME/flask-app'
        DEPLOY_SERVER = 'ec2-user@YOUR_EC2_IP'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git REPO_URL
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE}:latest .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                        docker push ${DOCKER_IMAGE}:latest
                    '''
                }
            }
        }

        stage('Deploy to Staging') {
            steps {
                sshagent(['deploy-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ${DEPLOY_SERVER} "
                        docker pull ${DOCKER_IMAGE}:latest &&
                        docker rm -f flask-container || true &&
                        docker run -d --name flask-container -p 5000:5000 ${DOCKER_IMAGE}:latest"
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution completed.'
        }
    }
}
