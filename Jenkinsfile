pipeline {
    agent any

    environment {
        IMAGE_NAME = "jitender12/nodedeploy"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Docker Check') {
            steps {
                bat 'docker --version'
            }
        }

        stage('Kubernetes Check') {
            steps {
                bat 'kubectl config current-context'
                bat 'kubectl get nodes'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %IMAGE_NAME%:%BUILD_NUMBER% .'
                bat 'docker tag %IMAGE_NAME%:%BUILD_NUMBER% %IMAGE_NAME%:latest'
            }
        }

        stage('Test Application') {
            steps {
                bat 'docker rm -f nodedeploy-test 2>nul || exit /b 0'
                bat 'docker run -d -p 3002:3000 --name nodedeploy-test %IMAGE_NAME%:%BUILD_NUMBER%'
                bat 'timeout /t 5 /nobreak'
                bat 'curl.exe http://localhost:3002/health'
                bat 'docker rm -f nodedeploy-test'
            }
        }

        stage('DockerHub Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    bat 'docker login -u %DOCKER_USERNAME% -p %DOCKER_PASSWORD%'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                bat 'docker push %IMAGE_NAME%:%BUILD_NUMBER%'
                bat 'docker push %IMAGE_NAME%:latest'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat 'kubectl set image deployment/nodedeploy nodedeploy=%IMAGE_NAME%:%BUILD_NUMBER%'
            }
        }

        stage('Verify Deployment') {
            steps {
                bat 'kubectl rollout status deployment/nodedeploy'
                bat 'kubectl get pods'
                bat 'kubectl get service nodedeploy-service'
            }
        }
    }
}