pipeline {
    agent any

    environment {
        APP_NAME = "devsecops-demo"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                bat 'pip install -r requirements.txt'
            }
        }

        stage('Run Tests') {
            steps {
                bat 'pytest test_app.py -v'
            }
        }

        stage('SonarQube SAST') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    bat "sonar-scanner -Dsonar.projectKey=devsecops-demo -Dsonar.sources=. -Dsonar.host.url=http://localhost:9000"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -t %APP_NAME%:%BUILD_NUMBER% ."
            }
        }

        stage('Trivy Security Scan') {
            steps {
                bat "C:\\Users\\Akshay-ragina-kothe\\AppData\\Local\\Microsoft\\WinGet\\Links\\trivy.exe image --exit-code 0 --severity HIGH,CRITICAL %APP_NAME%:%BUILD_NUMBER%"
            }
        }
    }

    post {
        success {
            echo "Pipeline SUCCESS"
        }
        failure {
            echo "Pipeline FAILED"
        }
    }
}