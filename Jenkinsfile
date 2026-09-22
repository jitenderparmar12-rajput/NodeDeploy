pipeline {
    agent any

    environment {
        IMAGE_NAME = "jitender12/nodedeploy"

        DOCKER = "C:\\Users\\JITENDER PARMAR\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe"

        KUBECTL = "C:\\Users\\JITENDER PARMAR\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\kubectl.exe"
        KUBECONFIG = "C:\\Users\\JITENDER PARMAR\\.kube\\config"
         }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Docker Check') {
            steps {
                bat '"%DOCKER%" --version'
            }
        }

        stage('Kubernetes Check') {
            steps {
                bat '"%KUBECTL%" config current-context'
                bat '"%KUBECTL%" get nodes'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat '"%DOCKER%" build -t %IMAGE_NAME%:%BUILD_NUMBER% .'
                bat '"%DOCKER%" tag %IMAGE_NAME%:%BUILD_NUMBER% %IMAGE_NAME%:latest'
            }
        }

        stage('Test Application') {
            steps {
                bat '"%DOCKER%" rm -f nodedeploy-test 2>nul || exit /b 0'

                bat '"%DOCKER%" run -d -p 3002:3000 --name nodedeploy-test %IMAGE_NAME%:%BUILD_NUMBER%'

                bat 'timeout /t 5 /nobreak'

                bat 'curl.exe http://localhost:3002/health'

                bat '"%DOCKER%" rm -f nodedeploy-test'
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
                    bat '"%DOCKER%" login -u %DOCKER_USERNAME% -p %DOCKER_PASSWORD%'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                bat '"%DOCKER%" push %IMAGE_NAME%:%BUILD_NUMBER%'
                bat '"%DOCKER%" push %IMAGE_NAME%:latest'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat '"%KUBECTL%" set image deployment/nodedeploy nodedeploy=%IMAGE_NAME%:%BUILD_NUMBER%'
            }
        }

        stage('Verify Deployment') {
            steps {
                bat '"%KUBECTL%" rollout status deployment/nodedeploy'
                bat '"%KUBECTL%" get pods'
                bat '"%KUBECTL%" get services'
            }
        }
    }

    post {
        success {
            echo 'NodeDeploy CI/CD Pipeline completed successfully!'
        }

        failure {
            echo 'NodeDeploy CI/CD Pipeline failed. Check the Console Output.'
        }
    }
}