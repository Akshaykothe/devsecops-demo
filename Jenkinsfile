pipeline {
    agent any

    environment {
        APP_NAME = "devsecops-demo"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Cloning repository..."
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                echo "Installing Python dependencies..."
                bat 'pip install -r requirements.txt'
            }
        }

        stage('Run Tests') {
            steps {
                echo "Running unit tests..."
                bat 'pytest test_app.py -v'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                bat "docker build -t %APP_NAME%:%BUILD_NUMBER% ."
            }
        }

        stage('Trivy Security Scan') {
            steps {
                echo "Scanning for vulnerabilities..."
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
```

**Open VS Code → Jenkinsfile → delete everything → paste above code → Ctrl+S**

Then run in CMD:
```
cd C:\Users\Akshay-ragina-kothe\Documents\devsecops-demo
git add .
git commit -m "Fix Trivy full path"
git push origin main