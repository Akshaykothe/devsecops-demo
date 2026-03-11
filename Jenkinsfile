pipeline {
    agent any

    environment {
        APP_NAME = "devsecops-demo"
        DOCKER_IMAGE = "${APP_NAME}:${BUILD_NUMBER}"
    }

    stages {

        stage('📥 Checkout') {
            steps {
                echo "Cloning repository..."
                checkout scm
            }
        }

        stage('🔧 Install Dependencies') {
            steps {
                echo "Installing Python dependencies..."
                sh 'pip install -r requirements.txt'
            }
        }

        stage('🧪 Run Tests') {
            steps {
                echo "Running unit tests..."
                sh 'pytest test_app.py -v'
            }
        }

        stage('🐳 Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('🔍 Trivy Security Scan') {
            steps {
                echo "Scanning for vulnerabilities..."
                sh "trivy image --exit-code 0 --severity HIGH,CRITICAL ${DOCKER_IMAGE}"
            }
        }

    }

    post {
        success {
            echo "✅ Pipeline SUCCESS - Build #${BUILD_NUMBER}"
        }
        failure {
            echo "❌ Pipeline FAILED - Check logs!"
        }
    }
}